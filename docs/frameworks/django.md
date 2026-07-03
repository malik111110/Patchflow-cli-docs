---
title: Django Pack
description: 7 rules for Python Django — SQLi, open redirect, XSS (mark_safe and safe filter), unsafe deserialization, CSRF, and SSRF
---

# Django Pack

## Overview

The Django pack covers Django views, the ORM (`raw`, `extra`, `cursor.execute`,
`RawSQL`), template engine `safe` filter, `mark_safe`, `pickle.loads`, the
`@csrf_exempt` decorator, and outbound HTTP via `requests`/`httpx`.

| Property | Value |
| --- | --- |
| Pack name | `django` |
| Language | Python |
| File extensions | `.py` |
| Template extensions | `.html`, `.jinja`, `.jinja2` |
| Rule count | 7 |
| Match modes | `pattern`, `template` |
| Maturity | `beta` (dev, pr, audit profiles) |

### Vulnerability categories covered

SQL Injection (CWE-89), Open Redirect (CWE-601), XSS (CWE-79), Unsafe
Deserialization (CWE-502), CSRF (CWE-352), SSRF (CWE-918).

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-DJANGO-SQLI-001 | Django SQLi: raw SQL built from request data | High | CWE-89 | pattern | Beta |
| PF-DJANGO-REDIRECT-001 | Django open redirect: redirect with request input | Medium | CWE-601 | pattern | Beta |
| PF-DJANGO-XSS-001 | Django XSS: mark_safe with request data | High | CWE-79 | pattern | Beta |
| PF-DJANGO-XSS-002 | Django template XSS: safe filter | High | CWE-79 | template | Beta |
| PF-DJANGO-DESER-001 | Django unsafe deserialization: pickle.loads on request data | Critical | CWE-502 | pattern | Beta |
| PF-DJANGO-CSRF-001 | Django CSRF: @csrf_exempt on view | Medium | CWE-352 | pattern | Beta |
| PF-DJANGO-SSRF-001 | Django SSRF: requests.get/httpx.get with user-controlled URL | High | CWE-918 | pattern | Beta |

## Sources

- `request.GET`
- `request.POST`
- `request.COOKIES`
- `request.headers`
- `request.body`
- `request.data`
- `request.FILES`
- `request.META`

## Sinks

| Function | Arg Index |
| --- | --- |
| `raw` | 0 |
| `extra` | -1 |
| `cursor.execute` | 0 |
| `RawSQL` | 0 |
| `redirect` | 0 |
| `mark_safe` | 0 |
| `pickle.loads` | 0 |
| `yaml.load` | 0 |
| `requests.get` | 0 |
| `httpx.get` | 0 |
| `subprocess.run` | 0 |

## Sanitizers

- `escape`
- `conditional_escape`
- `format_html`
- `url_has_allowed_host_and_scheme`
- `bleach.clean`
- `yaml.safe_load`

## Safe Patterns

| Pattern | Suppresses |
| --- | --- |
| `GET\|HEAD\|OPTIONS` | PF-DJANGO-CSRF-001 |

CSRF protection is not required for safe methods (`GET`, `HEAD`, `OPTIONS`), so
`@csrf_exempt` on read-only views is suppressed when only safe methods are
handled.

## Exclusions

The Django pack excludes the following paths to reduce false positives:

- `django/**`
- `docs/**`
- `tests/**`

## Usage

### Auto-detection

PatchFlow auto-detects Django from `manage.py`, `settings.py` with
`INSTALLED_APPS`, or `requirements.txt`/`pyproject.toml` containing `django`.

```bash
patchflow scan run
```

### Force-enable the Django pack

```bash
patchflow scan run --framework django
```

### List rules

```bash
patchflow rules list --framework django
```

### Explain a rule

```bash
patchflow explain --rule PF-DJANGO-SQLI-001
```

## Example Findings

### PF-DJANGO-SQLI-001 — raw SQL built from request data

```python
def search(request):
    q = request.GET.get("q", "")
    results = MyModel.objects.raw(
        "SELECT * FROM myapp_mymodel WHERE name LIKE '%%%s%%'" % q
    )
    return render(request, "results.html", {"results": results})
```

```text
PF-DJANGO-SQLI-001  High  CWE-89
  Django SQLi: raw SQL built from request data
  src/myapp/views.py:23
  Source: request.GET  Sink: raw
  Fix: use parameterized raw query — MyModel.objects.raw(
    "SELECT * FROM myapp_mymodel WHERE name LIKE %s", [f"%{q}%"])
```

### PF-DJANGO-XSS-001 — mark_safe with request data

```python
def profile(request):
    bio = request.POST.get("bio", "")
    return render(request, "profile.html", {"bio": mark_safe(bio)})
```

`mark_safe` marks a string as safe to render without escaping. Never apply it
to user input. Use `escape()` or `format_html()` instead, or simply pass the
raw string and let the template auto-escape.

### PF-DJANGO-XSS-002 — template safe filter

```html
<div>{{ user_comment|safe }}</div>
```

The `safe` filter disables auto-escaping for the variable. Remove the filter
to let Django escape the output, or sanitize with `bleach.clean` in the view.

### PF-DJANGO-CSRF-001 — @csrf_exempt on view

```python
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def update_settings(request):
    if request.method == "POST":
        # state-changing POST handler with CSRF disabled
        ...
```

```text
PF-DJANGO-CSRF-001  Medium  CWE-352
  Django CSRF: @csrf_exempt on view
  src/myapp/views.py:45
  Sink: @csrf_exempt
  Fix: remove @csrf_exempt; handle CSRF via Django middleware
```

### PF-DJANGO-SSRF-001 — requests.get with user-controlled URL

```python
import requests

def fetch_preview(request):
    url = request.GET.get("url")
    resp = requests.get(url)
    return HttpResponse(resp.content)
```

Validate the URL scheme and host against an allowlist, or use
`url_has_allowed_host_and_scheme` before making the request.

## Custom Extensions

Extend the Django pack with organization-specific sources, sinks, sanitizers,
and safe patterns:

```yaml
schema_version: "1.0"

framework_extensions:
  django:
    custom_sources:
      - function: "get_tenant_query"
    custom_sinks:
      - function: "legacy_sql.execute"
        cwe: "CWE-89"
        category: "sql_injection"
    custom_sanitizers:
      - function: "company_clean_html"
    safe_patterns:
      - pattern: "assert_tenant_owner"
        reason: "Tenant ownership check before data mutation"
```

```bash
patchflow rules validate .patchflow/rules.yaml
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full schema reference.
