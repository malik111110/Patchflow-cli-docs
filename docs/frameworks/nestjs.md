---
title: NestJS Pack
description: 4 rules for TypeScript NestJS — SQLi, SSRF, open redirect, and missing authorization on controllers
---

# NestJS Pack

## Overview

The NestJS pack detects vulnerabilities specific to the NestJS framework for TypeScript.
It understands NestJS controller annotations, request objects, and common data-access
and HTTP-client sinks.

- **Pack name:** `nestjs`
- **Language:** TypeScript
- **File extensions:** `.ts`
- **Template extensions:** none
- **Rules:** 4
- **Key vulnerability categories:** SQL Injection, Server-Side Request Forgery (SSRF),
  Open Redirect, Missing Authorization

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-NESTJS-SQLI-001 | NestJS SQLi: query built from controller input | High | CWE-89 | pattern | Beta |
| PF-NESTJS-SSRF-001 | NestJS SSRF: outbound request with controller input | High | CWE-918 | pattern | Beta |
| PF-NESTJS-REDIRECT-001 | NestJS open redirect: redirect with controller input | Medium | CWE-601 | pattern | Beta |
| PF-NESTJS-AUTH-001 | NestJS missing authorization: sensitive controller/route without auth guard | Medium | CWE-862 | pattern | Beta |

## Sources

The pack treats the following as untrusted user input:

- `@Query` (annotation)
- `@Param` (annotation)
- `@Body` (annotation)
- `@Headers` (annotation)
- `@Req` (annotation)
- `req.query`
- `req.body`
- `Request` (object)

## Sinks

| Function | Arg Index |
| --- | --- |
| `query` | 0 |
| `redirect` | 0 |
| `HttpService.get` | 0 |
| `HttpService.post` | 0 |
| `axios.get` | 0 |
| `exec` | 0 |
| `spawn` | 0 |

## Sanitizers

The following functions and mechanisms suppress findings when present:

- `encodeURIComponent`
- `isSafeRedirect`
- `allowlistedHost`
- `ValidationPipe`
- `Passport`

## Safe Patterns

- `@UseGuards` / `@Roles` — suppresses **PF-NESTJS-AUTH-001** when applied to a
  controller or route handler.

## Usage

### Auto-detection

NestJS is auto-detected when a `nest-cli.json` or a `@nestjs/core` dependency is
present in the project.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework nestjs
```

### Disable

```bash
patchflow scan run --disable-framework nestjs
```

### List rules

```bash
patchflow rules list --framework nestjs
```

### Explain a rule

```bash
patchflow explain --rule PF-NESTJS-SQLI-001
```

## Example Findings

### PF-NESTJS-SQLI-001 — SQL Injection

```typescript
import { Controller, Get, Query } from '@nestjs/common';
import { DatabaseService } from './database.service';

@Controller('users')
export class UsersController {
  constructor(private readonly db: DatabaseService) {}

  @Get('search')
  search(@Query('name') name: string) {
    // Vulnerable: user input concatenated into raw SQL
    return this.db.query(`SELECT * FROM users WHERE name = '${name}'`);
  }
}
```

**Expected finding:**

```
PF-NESTJS-SQLI-001  High  CWE-89
  users.controller.ts:10
  NestJS SQLi: query built from controller input
  Source: @Query('name')
  Sink:   this.db.query(...)
```

**Fixed version (parameterized query):**

```typescript
@Get('search')
search(@Query('name') name: string) {
  return this.db.query('SELECT * FROM users WHERE name = $1', [name]);
}
```

### PF-NESTJS-SSRF-001 — Server-Side Request Forgery

```typescript
import { Controller, Get, Query } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';

@Controller('fetch')
export class FetchController {
  constructor(private readonly http: HttpService) {}

  @Get()
  fetch(@Query('url') url: string) {
    // Vulnerable: outbound request to user-supplied URL
    return this.http.get(url);
  }
}
```

**Expected finding:**

```
PF-NESTJS-SSRF-001  High  CWE-918
  fetch.controller.ts:11
  NestJS SSRF: outbound request with controller input
  Source: @Query('url')
  Sink:   this.http.get(url)
```

### PF-NESTJS-REDIRECT-001 — Open Redirect

```typescript
@Get('redirect')
redirect(@Query('target') target: string, @Res() res: Response) {
  // Vulnerable: redirect to user-supplied target
  return res.redirect(target);
}
```

**Expected finding:**

```
PF-NESTJS-REDIRECT-001  Medium  CWE-601
  redirect.controller.ts:9
  NestJS open redirect: redirect with controller input
  Source: @Query('target')
  Sink:   res.redirect(target)
```

### PF-NESTJS-AUTH-001 — Missing Authorization

```typescript
@Controller('admin')
export class AdminController {
  @Get('settings')
  getSettings() {
    // Vulnerable: sensitive route without @UseGuards or @Roles
    return this.settingsService.findAll();
  }
}
```

**Expected finding:**

```
PF-NESTJS-AUTH-001  Medium  CWE-862
  admin.controller.ts:7
  NestJS missing authorization: sensitive controller/route without auth guard
```

**Fixed version:**

```typescript
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
@Roles('admin')
export class AdminController {
  @Get('settings')
  getSettings() {
    return this.settingsService.findAll();
  }
}
```

## Custom Extensions

You can add organization-specific sources, sinks, sanitizers, and safe patterns
to the NestJS pack. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for details.

```yaml
# .patchflow/framework-extensions.yml
framework: nestjs
extensions:
  sources:
    - "@Session"
  sinks:
    - function: "customHttpClient.get"
      argIndex: 0
  sanitizers:
    - "myAllowlistValidator"
```
