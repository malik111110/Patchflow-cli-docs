---
title: FastAPI Pack
description: 9 rules for Python FastAPI — SQLi, SSRF, open redirect, command injection, path traversal, XSS, missing auth, and taint-tracked SQLi/redirect
---

# FastAPI Pack

## Overview

The FastAPI pack covers FastAPI path operations, the `Request` object, dependency
injection parameters (`Query`, `Path`, `Header`, `Cookie`, `Body`), SQLAlchemy
`execute`/`text`, outbound HTTP via `requests`/`httpx`, `RedirectResponse`,
`subprocess`, `FileResponse`/`open`, and Jinja2 `safe` filter.

| Property | Value |
| --- | --- |
| Pack name | `fastapi` |
| Language | Python |
| File extensions | `.py` |
| Template extensions | `.html`, `.jinja`, `.jinja2` |
| Rule count | 9 |
| Match modes | `pattern`, `template`, `taint` |
| Maturity | `experimental` (audit profile only) |

### Vulnerability categories covered

SQL Injection (CWE-89), SSRF (CWE-918), Open Redirect (CWE-601), Command
Injection (CWE-78), Path Traversal (CWE-22), XSS (CWE-79), Missing
Authentication (CWE-862).

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-FASTAPI-SQLI-001 | FastAPI SQLi: execute with interpolated request data | High | CWE-89 | pattern | Experimental |
| PF-FASTAPI-SSRF-001 | FastAPI SSRF: outbound HTTP call with request-controlled URL | High | CWE-918 | pattern | Experimental |
| PF-FASTAPI-REDIRECT-001 | FastAPI open redirect: RedirectResponse with request input | Medium | CWE-601 | pattern | Experimental |
| PF-FASTAPI-CMDI-001 | FastAPI command injection: subprocess with request data | Critical | CWE-78 | pattern | Experimental |
| PF-FASTAPI-PATH-001 | FastAPI path traversal: FileResponse/open with request input | High | CWE-22 | pattern | Experimental |
| PF-FASTAPI-XSS-001 | FastAPI template XSS: Jinja safe filter | High | CWE-79 | template | Experimental |
| PF-FASTAPI-SQLI-002 | FastAPI SQLi (taint): request data reaches execute/text() | High | CWE-89 | taint | Experimental |
| PF-FASTAPI-REDIRECT-002 | FastAPI open redirect (taint): request data reaches RedirectResponse | Medium | CWE-601 | taint | Experimental |
| PF-FASTAPI-AUTH-001 | FastAPI missing auth: sensitive endpoint without Depends(auth) | Medium | CWE-862 | pattern | Experimental |

## Sources

- `request.query_params`
- `request.path_params`
- `request.headers`
- `request.cookies`
- `request.json`
- `request.form`
- `Query`
- `Path`
- `Header`
- `Cookie`
- `Body`

## Sinks

| Function | Arg Index |
| --- | --- |
| `execute` | 0 |
| `executemany` | 0 |
| `text` | 0 |
| `session.execute` | 0 |
| `requests.get` | 0 |
| `requests.post` | 0 |
| `httpx.get` | 0 |
| `httpx.post` | 0 |
| `RedirectResponse` | 0 |
| `subprocess.run` | 0 |
| `subprocess.Popen` | 0 |
| `subprocess.call` | 0 |
| `subprocess.check_output` | 0 |
| `FileResponse` | 0 |
| `open` | 0 |

## Sanitizers

- `html.escape`
- `markupsafe.escape`
- `urllib.parse.quote`
- `is_safe_url`
- `allow_redirect_host`
- `Path.resolve`
- `Path.is_relative_to`

## Safe Patterns

| Pattern | Suppresses |
| --- | --- |
| `Depends()` | PF-FASTAPI-AUTH-001 |
| `current_user\|get_current_user\|require_auth\|require_admin\|verify_token` | PF-FASTAPI-AUTH-001 |

When a sensitive endpoint declares a `Depends()` dependency that resolves to an
auth helper (`current_user`, `get_current_user`, `require_auth`,
`require_admin`, or `verify_token`), the missing-auth finding is suppressed.

## Usage

### Auto-detection

PatchFlow auto-detects FastAPI from `fastapi` in `requirements.txt`/
`pyproject.toml`, `FastAPI()` instantiation, or `from fastapi import`
references.

```bash
patchflow scan run
```

### Force-enable the FastAPI pack

```bash
patchflow scan run --framework fastapi
```

### List rules

```bash
patchflow rules list --framework fastapi
```

### Explain a rule

```bash
patchflow explain --rule PF-FASTAPI-SQLI-001
```

## Example Findings

### PF-FASTAPI-SQLI-001 — execute with interpolated request data

```python
from fastapi import FastAPI, Query
from sqlalchemy import text

app = FastAPI()

@app.get("/users")
def users(name: str = Query(...)):
    sql = f"SELECT * FROM users WHERE name = '{name}'"
    rows = session.execute(text(sql)).fetchall()
    return rows
```

```text
PF-FASTAPI-SQLI-001  High  CWE-89
  FastAPI SQLi: execute with interpolated request data
  src/app/routes.py:16
  Source: Query  Sink: session.execute
  Fix: use bind parameters — text("SELECT * FROM users WHERE name = :name"), {"name": name}
```

### PF-FASTAPI-CMDI-001 — subprocess with request data

```python
import subprocess
from fastapi import FastAPI, Body

app = FastAPI()

@app.post("/render")
def render(payload: dict = Body(...)):
    subprocess.run(f"wkhtmltopdf {payload['url']} out.pdf", shell=True)
    return {"status": "ok"}
```

Use `subprocess.run(["wkhtmltopdf", payload["url"], "out.pdf"])` (no
`shell=True`) and validate the URL against an allowlist.

### PF-FASTAPI-AUTH-001 — missing auth on sensitive endpoint

```python
@app.delete("/admin/users/{user_id}")
def delete_user(user_id: int):
    # no Depends(auth) — any caller can delete users
    users.delete(user_id)
    return {"deleted": user_id}
```

```text
PF-FASTAPI-AUTH-001  Medium  CWE-862
  FastAPI missing auth: sensitive endpoint without Depends(auth)
  src/app/admin.py:22
  Fix: add a dependency — def delete_user(user_id: int, user = Depends(require_admin)):
```

### PF-FASTAPI-PATH-001 — FileResponse with request input

```python
from fastapi import FastAPI, Query
from fastapi.responses import FileResponse

app = FastAPI()

@app.get("/files")
def files(name: str = Query(...)):
    return FileResponse(f"/var/data/{name}")
```

Resolve the path and verify it stays within the base directory:

```python
from pathlib import Path

BASE = Path("/var/data").resolve()

@app.get("/files")
def files(name: str = Query(...)):
    target = (BASE / name).resolve()
    if not target.is_relative_to(BASE):
        raise HTTPException(status_code=400, detail="Invalid path")
    return FileResponse(target)
```

### PF-FASTAPI-SQLI-002 — taint-tracked SQLi

```python
@app.get("/search")
def search(q: str = Query(...)):
    clause = "WHERE name LIKE '%" + q + "%'"
    rows = session.execute(text("SELECT * FROM products " + clause)).fetchall()
    return rows
```

The taint engine tracks `Query` → string concatenation → `text()` →
`session.execute`, flagging the flow even when the sink call does not directly
reference the source parameter.

## Custom Extensions

Extend the FastAPI pack with organization-specific sources, sinks, sanitizers,
and safe patterns:

```yaml
schema_version: "1.0"

framework_extensions:
  fastapi:
    custom_sources:
      - function: "get_tenant_query"
    custom_sinks:
      - function: "legacy_repo.find"
        cwe: "CWE-89"
        category: "sql_injection"
    custom_sanitizers:
      - function: "company_safe_redirect"
    safe_patterns:
      - pattern: "require_admin_dep"
        reason: "Admin dependency enforced on sensitive endpoints"
```

```bash
patchflow rules validate .patchflow/rules.yaml
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full schema reference.
