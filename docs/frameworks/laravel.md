---
title: Laravel Pack
description: 6 rules for PHP Laravel — SQLi, open redirect, Blade XSS, mass assignment, deserialization, and missing authentication
---

# Laravel Pack

## Overview

The Laravel pack provides framework-aware static analysis for Laravel
applications. It understands Laravel-specific sources (`request`, `$request->input`,
`Input::get`, `$_GET`), sinks (`DB::raw`, `redirect()->away`, `unserialize`,
`View::make`), and sanitizers (`e`, `htmlspecialchars`, `route`, `Validator`),
and it scans Blade templates for unescaped output.

| Property | Value |
| --- | --- |
| Pack name | `laravel` |
| Language | PHP |
| File extensions | `.php` |
| Template extensions | `.blade.php` |
| Rule count | 6 |
| Key coverage | SQL injection, open redirect, Blade XSS, mass assignment, unsafe deserialization, missing authentication |

Vulnerability categories covered:

- **CWE-89** SQL Injection — 1 rule
- **CWE-601** Open Redirect — 1 rule
- **CWE-79** Cross-Site Scripting (XSS) — 1 rule
- **CWE-915** Mass Assignment — 1 rule
- **CWE-502** Unsafe Deserialization — 1 rule
- **CWE-306** Missing Authentication — 1 rule

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-LARAVEL-SQLI-001 | Laravel SQLi: raw query built from request data | High | CWE-89 | pattern | Beta |
| PF-LARAVEL-REDIRECT-001 | Laravel open redirect: `away()` with request input | Medium | CWE-601 | pattern | Beta |
| PF-LARAVEL-XSS-001 | Laravel Blade XSS: unescaped output | High | CWE-79 | template | Beta |
| PF-LARAVEL-MASS-001 | Laravel mass assignment: `create` with all request fields | Medium | CWE-915 | pattern | Beta |
| PF-LARAVEL-DESER-001 | Laravel unsafe deserialization: `unserialize()` with user input | Critical | CWE-502 | pattern | Beta |
| PF-LARAVEL-AUTH-001 | Laravel missing authentication: sensitive route without auth middleware | Medium | CWE-306 | pattern | Beta |

## Sources

PatchFlow treats the following Laravel request inputs as taint sources:

- `request`
- `request()`
- `$request->input`
- `$request->query`
- `$request->get`
- `$request->all`
- `Input::get`
- `$_GET`
- `$_POST`
- `$_REQUEST`
- `$_COOKIE`

## Sinks

| Function | Arg Index |
| --- | --- |
| `DB::raw` | 0 |
| `DB::select` | 0 |
| `DB::statement` | 0 |
| `whereRaw` | 0 |
| `selectRaw` | 0 |
| `redirect()->away` | 0 |
| `redirect` | 0 |
| `unserialize` | 0 |
| `Storage::put` | 1 |
| `View::make` | -1 |
| `create` | 0 |

An arg index of `-1` means any argument; `0` means the first positional
argument is the tainted one. `Storage::put` taints the second argument
(content), hence index `1`.

## Sanitizers

Findings are suppressed when one of the following sanitizers wraps the tainted
data before it reaches a sink:

- `e(`
- `htmlspecialchars`
- `strip_tags`
- `route(`
- `url(`
- `Validator::make`
- `validator`
- `bcrypt`

## Safe Patterns

| Safe Pattern | Suppresses Rule |
| --- | --- |
| `->middleware(` | PF-LARAVEL-AUTH-001 |
| `auth:` | PF-LARAVEL-AUTH-001 |

Routes protected by middleware (e.g. `->middleware('auth')` or the `auth:`
middleware alias) are not flagged by the missing-authentication rule.

## Usage

### Auto-detection

PatchFlow auto-detects Laravel projects via `composer.json` / `artisan` and
activates the `laravel` pack automatically.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework laravel
```

### List Laravel rules

```bash
patchflow rules list --framework laravel
```

### Explain a specific rule

```bash
patchflow explain --rule PF-LARAVEL-SQLI-001
```

## Example Findings

### PF-LARAVEL-SQLI-001 — Raw query built from request data

```php
// app/Http/Controllers/SearchController.php
public function index(Request $request)
{
    $q = $request->input('q');
    $results = DB::select("SELECT * FROM products WHERE name LIKE '%{$q}%'"); // VULNERABLE
    return view('search', ['results' => $results]);
}
```

**Expected finding:**

```
PF-LARAVEL-SQLI-001  High  CWE-89
  app/Http/Controllers/SearchController.php:4
  Laravel SQLi: raw query built from request data
  User input ($request->input) interpolated into DB::select.
```

**Remediation** — use parameter bindings or the query builder:

```php
$results = DB::select("SELECT * FROM products WHERE name LIKE ?", ["%{$q}%"]);
// or
$results = Product::where('name', 'LIKE', "%{$q}%")->get();
```

### PF-LARAVEL-XSS-001 — Blade unescaped output

```blade
{{-- resources/views/profile.blade.php --}}
<h1>{!! $user->bio !!}</h1> {{-- VULNERABLE --}}
```

**Expected finding:**

```
PF-LARAVEL-XSS-001  High  CWE-79
  resources/views/profile.blade.php:1
  Laravel Blade XSS: unescaped output
  Blade unescaped output ({!! !!}) renders user data without escaping.
```

**Remediation** — use the default `{{ }}` escaping syntax, or `e()`:

```blade
<h1>{{ $user->bio }}</h1>
```

### PF-LARAVEL-REDIRECT-001 — Open redirect via `away()`

```php
// app/Http/Controllers/AuthController.php
public function login(Request $request)
{
    // ... authenticate ...
    return redirect()->away($request->input('return_to')); // VULNERABLE
}
```

**Remediation** — validate the URL against an allowlist:

```php
$returnTo = $request->input('return_to');
if (in_array($returnTo, config('app.allowed_redirects'), true)) {
    return redirect()->away($returnTo);
}
return redirect()->route('home');
```

### PF-LARAVEL-DESER-001 — `unserialize()` with user input

```php
// app/Http/Controllers/ImportController.php
public function import(Request $request)
{
    $data = unserialize($request->input('payload')); // VULNERABLE
    return $this->process($data);
}
```

**Remediation** — use `json_decode` instead of `unserialize`:

```php
$data = json_decode($request->input('payload'), true);
```

### PF-LARAVEL-MASS-001 — Mass assignment via `create`

```php
// app/Http/Controllers/UserController.php
public function store(Request $request)
{
    $user = User::create($request->all()); // VULNERABLE
    return response()->json($user);
}
```

**Remediation** — allowlist fields via `$fillable` or explicit input:

```php
$user = User::create($request->only(['name', 'email', 'password']));
```

### PF-LARAVEL-AUTH-001 — Sensitive route without auth middleware

```php
// routes/web.php
Route::get('/admin/dashboard', [AdminController::class, 'dashboard']); // VULNERABLE
```

**Remediation** — apply the auth middleware:

```php
Route::middleware('auth')->get('/admin/dashboard', [AdminController::class, 'dashboard']);
```

## Custom Extensions

You can extend the Laravel pack with organization-specific sources, sinks,
sanitizers, and safe patterns. For example, to register a custom sanitizer
used by your internal helpers:

```yaml
# .patchflow/framework-extensions.yml
pack: laravel
extensions:
  sanitizers:
    - App\Security\Sanitizer::clean
  safe_patterns:
    - pattern: "->middleware('auth:api')"
      suppresses: PF-LARAVEL-AUTH-001
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full extension schema and advanced examples.
