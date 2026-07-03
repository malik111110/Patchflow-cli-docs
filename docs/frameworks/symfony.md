---
title: Symfony Pack
description: 3 rules for PHP Symfony — Doctrine SQLi, open redirect, and Twig XSS
---

# Symfony Pack

## Overview

The Symfony pack provides framework-aware static analysis for Symfony
applications. It understands Symfony-specific sources
(`$request->query->get`, `$request->request->get`, `$request->headers->get`),
sinks (`createQuery`, `executeQuery`, `RedirectResponse`, `Response`), and
sanitizers (`setParameter`, `escape`, `UrlHelper`), and it scans Twig
templates for the `raw` filter.

| Property | Value |
| --- | --- |
| Pack name | `symfony` |
| Language | PHP |
| File extensions | `.php` |
| Template extensions | `.twig` |
| Rule count | 3 |
| Key coverage | Doctrine SQL injection, open redirect, Twig XSS |

Vulnerability categories covered:

- **CWE-89** SQL Injection — 1 rule
- **CWE-601** Open Redirect — 1 rule
- **CWE-79** Cross-Site Scripting (XSS) — 1 rule

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-SYMFONY-SQLI-001 | Symfony SQLi: Doctrine query built from request data | High | CWE-89 | pattern | Beta |
| PF-SYMFONY-REDIRECT-001 | Symfony open redirect: `RedirectResponse` with request input | Medium | CWE-601 | pattern | Beta |
| PF-SYMFONY-XSS-001 | Symfony Twig XSS: `raw` filter | High | CWE-79 | template | Beta |

## Sources

PatchFlow treats the following Symfony request inputs as taint sources:

- `$request->query->get`
- `$request->request->get`
- `$request->headers->get`
- `$request->cookies->get`

## Sinks

| Function | Arg Index |
| --- | --- |
| `createQuery` | 0 |
| `executeQuery` | 0 |
| `RedirectResponse` | 0 |
| `Response` | 0 |

An arg index of `0` means the first positional argument is the tainted one.

## Sanitizers

Findings are suppressed when one of the following sanitizers wraps the tainted
data before it reaches a sink:

- `setParameter`
- `escape`
- `htmlspecialchars`
- `UrlHelper`
- `isSafeRedirect`

## Safe Patterns

This pack does not currently define explicit safe-pattern suppressions.
Sanitizer coverage (above) is the primary suppression mechanism.

## Usage

### Auto-detection

PatchFlow auto-detects Symfony projects via `composer.json` (symfony/symfony
or symfony/framework-bundle) and the `bin/console` entrypoint, and activates
the `symfony` pack automatically.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework symfony
```

### List Symfony rules

```bash
patchflow rules list --framework symfony
```

### Explain a specific rule

```bash
patchflow explain --rule PF-SYMFONY-SQLI-001
```

## Example Findings

### PF-SYMFONY-SQLI-001 — Doctrine query built from request data

```php
// src/Controller/SearchController.php
class SearchController extends AbstractController
{
    public function index(Request $request): Response
    {
        $q = $request->query->get('q');
        $em = $this->getDoctrine()->getManager();
        $results = $em->createQuery("SELECT p FROM App\Entity\Product p WHERE p.name LIKE '%{$q}%'") // VULNERABLE
            ->getResult();
        return $this->render('search/index.html.twig', ['results' => $results]);
    }
}
```

**Expected finding:**

```
PF-SYMFONY-SQLI-001  High  CWE-89
  src/Controller/SearchController.php:7
  Symfony SQLi: Doctrine query built from request data
  User input ($request->query->get) interpolated into createQuery.
```

**Remediation** — use parameter binding via `setParameter`:

```php
$results = $em->createQuery("SELECT p FROM App\Entity\Product p WHERE p.name LIKE :q")
    ->setParameter('q', "%{$q}%")
    ->getResult();
```

### PF-SYMFONY-REDIRECT-001 — Open redirect via `RedirectResponse`

```php
// src/Controller/AuthController.php
class AuthController extends AbstractController
{
    public function login(Request $request): RedirectResponse
    {
        // ... authenticate ...
        return new RedirectResponse($request->query->get('return_to')); // VULNERABLE
    }
}
```

**Remediation** — validate the URL against an allowlist:

```php
$returnTo = $request->query->get('return_to');
$allowed = $this->getParameter('app.allowed_redirects');
if (in_array($returnTo, $allowed, true)) {
    return new RedirectResponse($returnTo);
}
return $this->redirectToRoute('home');
```

### PF-SYMFONY-XSS-001 — Twig `raw` filter

```twig
{# templates/profile/index.html.twig #}
<h1>{{ user.bio|raw }}</h1> {# VULNERABLE #}
```

**Expected finding:**

```
PF-SYMFONY-XSS-001  High  CWE-79
  templates/profile/index.html.twig:1
  Symfony Twig XSS: raw filter
  Twig raw filter renders user data without escaping.
```

**Remediation** — remove the `raw` filter and rely on Twig's default escaping:

```twig
<h1>{{ user.bio }}</h1>
```

## Custom Extensions

You can extend the Symfony pack with organization-specific sources, sinks,
sanitizers, and safe patterns. For example, to register a custom sanitizer
used by your internal helpers:

```yaml
# .patchflow/framework-extensions.yml
pack: symfony
extensions:
  sanitizers:
    - App\Security\Twig\Sanitizer::clean
  safe_patterns:
    - pattern: "isSafeRedirect("
      suppresses: PF-SYMFONY-REDIRECT-001
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full extension schema and advanced examples.
