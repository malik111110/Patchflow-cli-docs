---
title: Gin Pack
description: 4 rules for Go Gin — SQLi, open redirect, XSS, and command injection from request data
---

# Gin Pack

## Overview

The Gin pack detects vulnerabilities specific to the Gin web framework for Go.
It understands Gin's context-based request accessors (`c.Query`, `c.Param`,
`c.PostForm`, `c.GetHeader`) and the common database, response, and OS-command
sinks.

- **Pack name:** `gin`
- **Language:** Go
- **File extensions:** `.go`
- **Template extensions:** none
- **Rules:** 4
- **Key vulnerability categories:** SQL Injection, Open Redirect, Cross-Site
  Scripting (XSS), Command Injection

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-GIN-SQLI-001 | Gin SQLi: database query built from request data | High | CWE-89 | pattern | Beta |
| PF-GIN-REDIRECT-001 | Gin open redirect: c.Redirect with request input | Medium | CWE-601 | pattern | Beta |
| PF-GIN-XSS-001 | Gin XSS: c.String with request input | High | CWE-79 | pattern | Beta |
| PF-GIN-CMDI-001 | Gin command injection: exec.Command with request data | Critical | CWE-78 | pattern | Beta |

## Sources

The pack treats the following as untrusted user input:

- `c.Query`
- `c.Param`
- `c.PostForm`
- `c.GetHeader`
- `c.Request.URL.Query`

## Sinks

| Function | Arg Index |
| --- | --- |
| `db.Raw` | 0 |
| `db.Exec` | 0 |
| `c.Redirect` | 1 |
| `c.String` | 1 |
| `c.File` | 0 |
| `exec.Command` | -1 |

## Sanitizers

The following functions and mechanisms suppress findings when present:

- `html.EscapeString`
- `url.QueryEscape`
- `filepath.Clean`
- `isSafeRedirect`

## Safe Patterns

- `html.EscapeString` — suppresses **PF-GIN-XSS-001**
- `url.QueryEscape` — suppresses **PF-GIN-REDIRECT-001** and **PF-GIN-XSS-001**
- `filepath.Clean` — suppresses path-related findings
- `isSafeRedirect` — suppresses **PF-GIN-REDIRECT-001**
- Parameterized queries (`db.Raw` with `?` placeholders and args) suppress
  **PF-GIN-SQLI-001**

## Usage

### Auto-detection

Gin is auto-detected when a `gin-gonic/gin` dependency is present in `go.mod`.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework gin
```

### Disable

```bash
patchflow scan run --disable-framework gin
```

### List rules

```bash
patchflow rules list --framework gin
```

### Explain a rule

```bash
patchflow explain --rule PF-GIN-SQLI-001
```

## Example Findings

### PF-GIN-SQLI-001 — SQL Injection

```go
package main

import (
    "github.com/gin-gonic/gin"
    "gorm.io/gorm"
)

func searchUsers(c *gin.Context, db *gorm.DB) {
    name := c.Query("name")
    // Vulnerable: request data concatenated into raw SQL
    var users []User
    db.Raw("SELECT * FROM users WHERE name = '" + name + "'").Scan(&users)
    c.JSON(200, users)
}
```

**Expected finding:**

```
PF-GIN-SQLI-001  High  CWE-89
  handlers.go:14
  Gin SQLi: database query built from request data
  Source: c.Query("name")
  Sink:   db.Raw(...)
```

**Fixed version (parameterized query):**

```go
func searchUsers(c *gin.Context, db *gorm.DB) {
    name := c.Query("name")
    var users []User
    db.Raw("SELECT * FROM users WHERE name = ?", name).Scan(&users)
    c.JSON(200, users)
}
```

### PF-GIN-REDIRECT-001 — Open Redirect

```go
func redirect(c *gin.Context) {
    target := c.Query("url")
    // Vulnerable: redirect to user-supplied URL
    c.Redirect(302, target)
}
```

**Expected finding:**

```
PF-GIN-REDIRECT-001  Medium  CWE-601
  handlers.go:8
  Gin open redirect: c.Redirect with request input
  Source: c.Query("url")
  Sink:   c.Redirect(302, target)
```

**Fixed version:**

```go
func redirect(c *gin.Context) {
    target := c.Query("url")
    if !isSafeRedirect(target) {
        c.String(400, "invalid redirect")
        return
    }
    c.Redirect(302, target)
}
```

### PF-GIN-XSS-001 — Cross-Site Scripting

```go
func greet(c *gin.Context) {
    name := c.Query("name")
    // Vulnerable: request data written directly to HTML response
    c.String(200, "<h1>Hello %s</h1>", name)
}
```

**Expected finding:**

```
PF-GIN-XSS-001  High  CWE-79
  handlers.go:8
  Gin XSS: c.String with request input
  Source: c.Query("name")
  Sink:   c.String(200, ...)
```

**Fixed version:**

```go
import "html"

func greet(c *gin.Context) {
    name := c.Query("name")
    c.String(200, "<h1>Hello %s</h1>", html.EscapeString(name))
}
```

### PF-GIN-CMDI-001 — Command Injection

```go
import "os/exec"

func ping(c *gin.Context) {
    host := c.Query("host")
    // Vulnerable: request data in exec.Command
    out, _ := exec.Command("ping", "-c", "1", host).Output()
    c.String(200, string(out))
}
```

**Expected finding:**

```
PF-GIN-CMDI-001  Critical  CWE-78
  handlers.go:10
  Gin command injection: exec.Command with request data
  Source: c.Query("host")
  Sink:   exec.Command(...)
```

**Fixed version (validate input against an allowlist):**

```go
import "regexp"

var hostRe = regexp.MustCompile(`^[a-zA-Z0-9.\-]+$`)

func ping(c *gin.Context) {
    host := c.Query("host")
    if !hostRe.MatchString(host) {
        c.String(400, "invalid host")
        return
    }
    out, _ := exec.Command("ping", "-c", "1", host).Output()
    c.String(200, string(out))
}
```

## Custom Extensions

You can add organization-specific sources, sinks, sanitizers, and safe patterns
to the Gin pack. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for details.

```yaml
# .patchflow/framework-extensions.yml
framework: gin
extensions:
  sources:
    - "c.Cookie"
  sinks:
    - function: "customDB.Query"
      argIndex: 0
  sanitizers:
    - "validateHost"
```
