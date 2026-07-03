---
title: ASP.NET Pack
description: 8 rules for C# ASP.NET Core — SQLi, open redirect, XSS, deserialization, command injection, and path traversal
---

# ASP.NET Pack

## Overview

The ASP.NET pack detects vulnerabilities specific to ASP.NET Core applications
written in C#. It understands ASP.NET Core request objects, model-binding
annotations, Entity Framework raw SQL APIs, and HTML helper methods.

- **Pack name:** `aspnet`
- **Language:** C#
- **File extensions:** `.cs`
- **Template extensions:** none (Razor templates are covered by the
  [Razor pack](./razor))
- **Rules:** 8
- **Key vulnerability categories:** SQL Injection, Open Redirect, Cross-Site
  Scripting (XSS), Unsafe Deserialization, Command Injection, Path Traversal

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-ASPNET-SQLI-001 | ASP.NET Core SQLi: raw SQL built from request data | High | CWE-89 | pattern | Beta |
| PF-ASPNET-REDIRECT-001 | ASP.NET Core open redirect: Redirect with request input | Medium | CWE-601 | pattern | Beta |
| PF-ASPNET-XSS-001 | ASP.NET Core XSS: raw HTML response from request data | High | CWE-79 | pattern | Beta |
| PF-ASPNET-SQLI-002 | ASP.NET Core SQLi: FromSqlRaw with string interpolation of user input | High | CWE-89 | pattern | Beta |
| PF-ASPNET-XSS-002 | ASP.NET Core XSS: @Html.Raw with user-controlled data | High | CWE-79 | pattern | Beta |
| PF-ASPNET-DESER-001 | ASP.NET Core deserialization: BinaryFormatter.Deserialize with user input | Critical | CWE-502 | pattern | Experimental |
| PF-ASPNET-CMDI-001 | ASP.NET Core command injection: Process.Start with request data | Critical | CWE-78 | pattern | Experimental |
| PF-ASPNET-PATH-001 | ASP.NET Core path traversal: Path.Combine with user input | High | CWE-22 | pattern | Experimental |

## Sources

The pack treats the following as untrusted user input:

- `Request.Query`
- `Request.Form`
- `Request.Headers`
- `HttpContext.Request.Query`
- `Request.RouteValues`
- `RouteData.Values`
- `[FromQuery]` (annotation)
- `[FromBody]` (annotation)

## Sinks

| Function | Arg Index |
| --- | --- |
| `FromSqlRaw` | 0 |
| `ExecuteSqlRaw` | 0 |
| `SqlCommand` | 0 |
| `Redirect` | 0 |
| `HtmlString` | 0 |
| `Content` | 0 |
| `BinaryFormatter.Deserialize` | 0 |
| `Process.Start` | 0 |
| `Path.Combine` | 1 |

## Sanitizers

The following functions and mechanisms suppress findings when present:

- `HtmlEncoder.Default.Encode`
- `WebUtility.HtmlEncode`
- `Url.IsLocalUrl`
- `LocalRedirect`
- `FromSqlInterpolated` / `ExecuteSqlInterpolated` (regex — parameterized SQL)
- `new SqlParameter` (regex — parameterized query)
- `Path.GetFullPath`
- `JsonSerializer.Deserialize`

## Safe Patterns

- `FromSqlInterpolated` / `ExecuteSqlInterpolated` — suppresses
  **PF-ASPNET-SQLI-001** and **PF-ASPNET-SQLI-002** (safe interpolated SQL)
- `new SqlParameter` — suppresses **PF-ASPNET-SQLI-001** (parameterized query)
- `Url.IsLocalUrl` / `LocalRedirect` — suppresses **PF-ASPNET-REDIRECT-001**
- `HtmlEncoder.Default.Encode` / `WebUtility.HtmlEncode` — suppresses
  **PF-ASPNET-XSS-001** and **PF-ASPNET-XSS-002**
- `Path.GetFullPath` — suppresses **PF-ASPNET-PATH-001**
- `JsonSerializer.Deserialize` — suppresses **PF-ASPNET-DESER-001** (safe
  deserialization alternative)

## Usage

### Auto-detection

ASP.NET Core is auto-detected when a `.csproj` file referencing
`Microsoft.AspNetCore.*` packages is present.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework aspnet
```

### Disable

```bash
patchflow scan run --disable-framework aspnet
```

### List rules

```bash
patchflow rules list --framework aspnet
```

### Explain a rule

```bash
patchflow explain --rule PF-ASPNET-SQLI-001
```

## Example Findings

### PF-ASPNET-SQLI-001 — Raw SQL with Request Data

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Data.SqlClient;

public class UsersController : Controller
{
    public IActionResult Search()
    {
        var name = Request.Query["name"];
        // Vulnerable: request data concatenated into raw SQL
        var cmd = new SqlCommand(
            $"SELECT * FROM Users WHERE Name = '{name}'",
            connection);
        return Json(cmd.ExecuteReader());
    }
}
```

**Expected finding:**

```
PF-ASPNET-SQLI-001  High  CWE-89
  UsersController.cs:12
  ASP.NET Core SQLi: raw SQL built from request data
  Source: Request.Query["name"]
  Sink:   new SqlCommand(...)
```

**Fixed version (parameterized query):**

```csharp
var cmd = new SqlCommand("SELECT * FROM Users WHERE Name = @name", connection);
cmd.Parameters.Add(new SqlParameter("@name", name));
```

### PF-ASPNET-SQLI-002 — FromSqlRaw with Interpolation

```csharp
public IActionResult GetUsers()
{
    var role = Request.Query["role"];
    // Vulnerable: string interpolation of user input in FromSqlRaw
    var users = _context.Users
        .FromSqlRaw($"SELECT * FROM Users WHERE Role = '{role}'")
        .ToList();
    return Json(users);
}
```

**Expected finding:**

```
PF-ASPNET-SQLI-002  High  CWE-89
  UsersController.cs:11
  ASP.NET Core SQLi: FromSqlRaw with string interpolation of user input
  Source: Request.Query["role"]
  Sink:   FromSqlRaw(...)
```

**Fixed version (FromSqlInterpolated):**

```csharp
var users = _context.Users
    .FromSqlInterpolated($"SELECT * FROM Users WHERE Role = {role}")
    .ToList();
```

### PF-ASPNET-REDIRECT-001 — Open Redirect

```csharp
public IActionResult Login(string returnUrl)
{
    // Vulnerable: redirect to user-supplied URL
    return Redirect(returnUrl);
}
```

**Expected finding:**

```
PF-ASPNET-REDIRECT-001  Medium  CWE-601
  AccountController.cs:8
  ASP.NET Core open redirect: Redirect with request input
  Source: returnUrl (action parameter)
  Sink:   Redirect(returnUrl)
```

**Fixed version:**

```csharp
public IActionResult Login(string returnUrl)
{
    if (Url.IsLocalUrl(returnUrl))
        return Redirect(returnUrl);
    return RedirectToAction("Index", "Home");
}
```

### PF-ASPNET-XSS-001 — Raw HTML Response

```csharp
public IActionResult Greet()
{
    var name = Request.Query["name"];
    // Vulnerable: unsanitized request data in HTML response
    return Content($"<h1>Hello {name}</h1>", "text/html");
}
```

**Expected finding:**

```
PF-ASPNET-XSS-001  High  CWE-79
  HomeController.cs:9
  ASP.NET Core XSS: raw HTML response from request data
  Source: Request.Query["name"]
  Sink:   Content(...)
```

### PF-ASPNET-DESER-001 — Unsafe Deserialization

```csharp
using System.Runtime.Serialization.Formatters.Binary;

public IActionResult Import()
{
    var data = Request.Body;
    // Vulnerable: BinaryFormatter.Deserialize on user input
    var formatter = new BinaryFormatter();
    var obj = formatter.Deserialize(data);
    return Ok(obj);
}
```

**Expected finding:**

```
PF-ASPNET-DESER-001  Critical  CWE-502
  ImportController.cs:11
  ASP.NET Core deserialization: BinaryFormatter.Deserialize with user input
  Source: Request.Body
  Sink:   formatter.Deserialize(data)
```

### PF-ASPNET-CMDI-001 — Command Injection

```csharp
using System.Diagnostics;

public IActionResult Ping()
{
    var host = Request.Query["host"];
    // Vulnerable: request data in Process.Start
    var psi = new ProcessStartInfo("ping", host);
    var proc = Process.Start(psi);
    return Ok(proc.StandardOutput.ReadToEnd());
}
```

**Expected finding:**

```
PF-ASPNET-CMDI-001  Critical  CWE-78
  NetworkController.cs:11
  ASP.NET Core command injection: Process.Start with request data
  Source: Request.Query["host"]
  Sink:   Process.Start(psi)
```

### PF-ASPNET-PATH-001 — Path Traversal

```csharp
using System.IO;

public IActionResult Download(string filename)
{
    // Vulnerable: user input in Path.Combine
    var path = Path.Combine(_basePath, filename);
    return PhysicalFile(path, "application/octet-stream");
}
```

**Expected finding:**

```
PF-ASPNET-PATH-001  High  CWE-22
  FileController.cs:9
  ASP.NET Core path traversal: Path.Combine with user input
  Source: filename (action parameter)
  Sink:   Path.Combine(_basePath, filename)
```

**Fixed version:**

```csharp
var path = Path.GetFullPath(Path.Combine(_basePath, filename));
if (!path.StartsWith(_basePath))
    return BadRequest("Invalid path");
return PhysicalFile(path, "application/octet-stream");
```

## Custom Extensions

You can add organization-specific sources, sinks, sanitizers, and safe patterns
to the ASP.NET pack. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for details.

```yaml
# .patchflow/framework-extensions.yml
framework: aspnet
extensions:
  sources:
    - "Request.Cookies"
  sinks:
    - function: "Dapper.SqlMapper.Query"
      argIndex: 0
  sanitizers:
    - "AntiXssEncoder.Encode"
```
