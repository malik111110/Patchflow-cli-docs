---
title: Echo Pack
description: 3 rules for Go Echo — SQLi, open redirect, and XSS from request data
---

# Echo Pack

## Overview

The Echo pack detects vulnerabilities specific to the Echo web framework for Go.
It understands Echo's context-based request accessors (`c.QueryParam`, `c.Param`,
`c.FormValue`, `c.Request().Header.Get`) and the common database, response, and
OS-command sinks.

- **Pack name:** `echo`
- **Language:** Go
- **File extensions:** `.go`
- **Template extensions:** none
- **Rules:** 3
- **Key vulnerability categories:** SQL Injection, Open Redirect, Cross-Site
  Scripting (XSS)

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-ECHO-SQLI-001 | Echo SQLi: database query built from request data | High | CWE-89 | pattern | Beta |
| PF-ECHO-REDIRECT-001 | Echo open redirect: c.Redirect with request input | Medium | CWE-601 | pattern | Beta |
| PF-ECHO-XSS-001 | Echo XSS: c.HTML with request input | High | CWE-79 | pattern | Beta |

## Sources

The pack treats the following as untrusted user input:

- `c.QueryParam`
- `c.Param`
- `c.FormValue`
- `c.Request().Header.Get`

## Sinks

| Function | Arg Index |
| --- | --- |
| `db.Raw` | 0 |
| `db.Exec` | 0 |
| `c.Redirect` | 1 |
| `c.HTML` | 1 |
| `c.File` | 0 |
| `exec.Command` | -1 |

## Sanitizers

The following functions and mechanisms suppress findings when present:

- `html.EscapeString`
- `url.QueryEscape`
- `filepath.Clean`
- `isSafeRedirect`

## Safe Patterns

- `html.EscapeString` — suppresses **PF-ECHO-XSS-001**
- `url.QueryEscape` — suppresses **PF-ECHO-REDIRECT-001** and **PF-ECHO-XSS-001**
- `filepath.Clean` — suppresses path-related findings
- `isSafeRedirect` — suppresses **PF-ECHO-REDIRECT-001**
- Parameterized queries (`db.Raw` with `?` placeholders and args) suppress
  **PF-ECHO-SQLI-001**

## Usage

### Auto-detection

Echo is auto-detected when a `labstack/echo` dependency is present in `go.mod`.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework echo
```

### Disable

```bash
patchflow scan run --disable-framework echo
```

### List rules

```bash
patchflow rules list --framework echo
```

### Explain a rule

```bash
patchflow explain --rule PF-ECHO-SQLI-001
```

## Example Findings

### PF-ECHO-SQLI-001 — SQL Injection

```go
package main

import (
    "github.com/labstack/echo/v4"
    "net/http"
    "gorm.io/gorm"
)

func searchUsers(c echo.Context, db *gorm.DB) error {
    name := c.QueryParam("name")
    // Vulnerable: request data concatenated into raw SQL
    var users []User
    db.Raw("SELECT * FROM users WHERE name = '" + name + "'").Scan(&users)
    return c.JSON(http.StatusOK, users)
}
```

**Expected finding:**

```
PF-ECHO-SQLI-001  High  CWE-89
  handlers.go:14
  Echo SQLi: database query built from request data
  Source: c.QueryParam("name")
  Sink:   db.Raw(...)
```

**Fixed version (parameterized query):**

```go
func searchUsers(c echo.Context, db *gorm.DB) error {
    name := c.QueryParam("name")
    var users []User
    db.Raw("SELECT * FROM users WHERE name = ?", name).Scan(&users)
    return c.JSON(http.StatusOK, users)
}
```

### PF-ECHO-REDIRECT-001 — Open Redirect

```go
func redirect(c echo.Context) error {
    target := c.QueryParam("url")
    // Vulnerable: redirect to user-supplied URL
    return c.Redirect(http.StatusFound, target)
}
```

**Expected finding:**

```
PF-ECHO-REDIRECT-001  Medium  CWE-601
  handlers.go:8
  Echo open redirect: c.Redirect with request input
  Source: c.QueryParam("url")
  Sink:   c.Redirect(302, target)
```

**Fixed version:**

```go
func redirect(c echo.Context) error {
    target := c.QueryParam("url")
    if !isSafeRedirect(target) {
        return c.String(http.StatusBadRequest, "invalid redirect")
    }
    return c.Redirect(http.StatusFound, target)
}
```

### PF-ECHO-XSS-001 — Cross-Site Scripting

```go
func greet(c echo.Context) error {
    name := c.QueryParam("name")
    // Vulnerable: request data written directly to HTML response
    return c.HTML(http.StatusOK, "<h1>Hello "+name+"</h1>")
}
```

**Expected finding:**

```
PF-ECHO-XSS-001  High  CWE-79
  handlers.go:8
  Echo XSS: c.HTML with request input
  Source: c.QueryParam("name")
  Sink:   c.HTML(200, ...)
```

**Fixed version:**

```go
import "html"

func greet(c echo.Context) error {
    name := c.QueryParam("name")
    return c.HTML(http.StatusOK, "<h1>Hello "+html.EscapeString(name)+"</h1>")
}
```

## Custom Extensions

You can add organization-specific sources, sinks, sanitizers, and safe patterns
to the Echo pack. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for details.

```yaml
# .patchflow/framework-extensions.yml
framework: echo
extensions:
  sources:
    - "c.Cookie"
  sinks:
    - function: "customDB.Query"
      argIndex: 0
  sanitizers:
    - "validateHost"
```
