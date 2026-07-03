---
title: Flask Pack
description: 9 rules for Python Flask — SQLi, SSRF, open redirect, XSS, SSTI, path traversal, debug mode, and taint-tracked SQLi/SSRF
---

# Flask Pack

## Overview

The Flask pack covers Flask views, the `request` proxy, SQLAlchemy/DB-API
`execute`, outbound HTTP via `requests`/`httpx`, `redirect`, Jinja2 `safe`
filter and `Markup`, `render_template_string` (SSTI), `send_file`/
`send_from_directory`, and debug/secret-key configuration.

| Property | Value |
| --- | --- |
| Pack name | `flask` |
| Language | Python |
| File extensions | `.py` |
| Template extensions | `.html`, `.jinja`, `.jinja2` |
| Rule count | 9 |
| Match modes | `pattern`, `template`, `taint` |
| Maturity | `beta` / `experimental` (taint rules are experimental) |

### Vulnerability categories covered

SQL Injection (CWE-89), SSRF (CWE-918), Open Redirect (CWE-601), XSS (CWE-79),
SSTI (CWE-94), Path Traversal (CWE-22), Debug Mode / Hardcoded Secret
(CWE-489).

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-FLASK-SQLI-001 | Flask SQLi: execute with request data | High | CWE-89 | pattern | Beta |
| PF-FLASK-SSRF-001 | Flask SSRF: outbound request with request-controlled URL | High | CWE-918 | pattern | Beta |
| PF-FLASK-REDIRECT-001 | Flask open redirect: redirect with request input | Medium | CWE-601 | pattern | Beta |
| PF-FLASK-XSS-001 | Flask template XSS: Jinja safe filter | High | CWE-79 | template | Beta |
| PF-FLASK-SSTI-001 | Flask SSTI: render_template_string with dynamic template | High | CWE-94 | pattern | Beta |
| PF-FLASK-PATH-001 | Flask path traversal: send_file/open with request input | High | CWE-22 | pattern | Beta |
| PF-FLASK-CONFIG-001 | Flask debug mode enabled or hardcoded secret key | Medium | CWE-489 | pattern | Beta |
| PF-FLASK-SQLI-002 | Flask SQLi: request data flows to raw SQL (taint-tracked) | High | CWE-89 | taint | Experimental |
| PF-FLASK-SSRF-002 | Flask SSRF: request data flows to outbound HTTP (taint-tracked) | High | CWE-918 | taint | Experimental |

## Sources

- `request.args`
- `request.form`
- `request.values`
- `request.headers`
- `request.cookies`
- `request.json`
- `request.get_json`
- `request.files`
- `request.data`

## Sinks

| Function | Arg Index |
| --- | --- |
| `cursor.execute` | 0 |
| `session.execute` | 0 |
| `db.session.execute` | 0 |
| `text` | 0 |
| `execute` | 0 |
| `requests.get` | 0 |
| `requests.post` | 0 |
| `httpx.get` | 0 |
| `httpx.post` | 0 |
| `redirect` | 0 |
| `Markup` | 0 |
| `render_template_string` | 0 |
| `send_file` | 0 |
| `send_from_directory` | 0 |
| `open` | 0 |
| `subprocess.run` | 0 |
| `os.system` | 0 |

## Sanitizers

- `escape`
- `markupsafe.escape`
- `html.escape`
- `url_has_allowed_host_and_scheme`
- `is_safe_url`

## Safe Patterns

| Pattern | Suppresses |
| --- | --- |
| `secure_filename` | PF-FLASK-PATH-001 |
| `os.environ\|getenv\|config.from_envvar` | PF-FLASK-CONFIG-001 |
| `:\w+` named parameter | PF-FLASK-SQLI-002 |
| `bindparam` | PF-FLASK-SQLI-002 |

Named parameters (`:name`) and `bindparam()` indicate parameterized queries,
suppressing the taint-tracked SQLi finding.

## Usage

### Auto-detection

PatchFlow auto-detects Flask from `app = Flask(__name__)`, `flask` in
`requirements.txt`/`pyproject.toml`, or `from flask import` references.

```bash
patchflow scan run
```

### Force-enable the Flask pack

```bash
patchflow scan run --framework flask
```

### List rules

```bash
patchflow rules list --framework flask
```

### Explain a rule

```bash
patchflow explain --rule PF-FLASK-SSTI-001
```

## Example Findings

### PF-FLASK-SQLI-001 — execute with request data

```python
from flask import request
from sqlalchemy import text

@app.route("/users")
def users():
    name = request.args.get("name", "")
    result = db.session.execute(
        text(f"SELECT * FROM users WHERE name = '{name}'")
    )
    return jsonify([r._asdict() for r in result])
```

```text
PF-FLASK-SQLI-001  High  CWE-89
  Flask SQLi: execute with request data
  src/app/views.py:18
  Source: request.args  Sink: db.session.execute
  Fix: use bind parameters — text("SELECT * FROM users WHERE name = :name"), {"name": name}
```

### PF-FLASK-SSTI-001 — render_template_string with dynamic template

```python
@app.route("/greet")
def greet():
    name = request.args.get("name", "")
    return render_template_string(f"<h1>Hello {name}</h1>")
```

`render_template_string` compiles and executes the template string, allowing
`{{ }}` and `{% %}` expressions to run arbitrary Python. Use
`render_template` with a static file and pass `name` as a context variable
instead.

### PF-FLASK-PATH-001 — send_file with request input

```python
from flask import send_file, request

@app.route("/download")
def download():
    filename = request.args.get("file", "")
    return send_file(f"/var/uploads/{filename}", as_attachment=True)
```

Use `secure_filename` and resolve against a fixed base directory, then verify
the resolved path stays within it.

### PF-FLASK-CONFIG-001 — debug mode enabled or hardcoded secret key

```python
app = Flask(__name__)
app.config["DEBUG"] = True
app.config["SECRET_KEY"] = "super-secret-hardcoded-key"
```

```text
PF-FLASK-CONFIG-001  Medium  CWE-489
  Flask debug mode enabled or hardcoded secret key
  src/app.py:14
  Fix: load SECRET_KEY from environment — app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]
        disable debug in production — app.run(debug=False)
```

### PF-FLASK-SQLI-002 — taint-tracked SQLi

```python
@app.route("/search")
def search():
    q = request.args.get("q")
    sql = "SELECT * FROM products WHERE name LIKE '%" + q + "%'"
    rows = db.session.execute(text(sql)).fetchall()
    return jsonify(rows)
```

The taint engine tracks `request.args` → string concatenation → `text()` →
`db.session.execute`, flagging the flow even though the sink call does not
directly reference the source.

## Custom Extensions

Extend the Flask pack with organization-specific sources, sinks, sanitizers,
and safe patterns:

```yaml
schema_version: "1.0"

framework_extensions:
  flask:
    custom_sources:
      - function: "get_tenant_args"
    custom_sinks:
      - function: "legacy_db.run"
        cwe: "CWE-89"
        category: "sql_injection"
    custom_sanitizers:
      - function: "company_safe_url"
    safe_patterns:
      - pattern: "require_tenant_scope"
        reason: "Tenant scope enforcement before query execution"
```

```bash
patchflow rules validate .patchflow/rules.yaml
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full schema reference.
