---
title: GraphQL Pack
description: 5 rules for Python GraphQL — SQLi, SSRF, path traversal, IDOR, and DoS in resolvers
---

# GraphQL Pack

## Overview

The GraphQL pack detects vulnerabilities in Python GraphQL resolver functions.
It understands GraphQL resolver context objects, argument passing, and the common
data-access and HTTP-client sinks used inside resolvers.

- **Pack name:** `graphql`
- **Language:** Python
- **File extensions:** `.py`
- **Template extensions:** none
- **Rules:** 5
- **Key vulnerability categories:** SQL Injection, Server-Side Request Forgery (SSRF),
  Path Traversal, IDOR / Broken Access Control, Denial of Service

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-GRAPHQL-SQLI-001 | GraphQL SQLi: resolver args flow to raw SQL (text/execute) | High | CWE-89 | taint | Beta |
| PF-GRAPHQL-SSRF-001 | GraphQL SSRF: resolver args flow to outbound HTTP request | High | CWE-918 | taint | Beta |
| PF-GRAPHQL-PATH-001 | GraphQL path traversal: resolver args flow to file operations | High | CWE-22 | taint | Beta |
| PF-GRAPHQL-AUTH-001 | GraphQL authorization: resolver fetches object by id without ownership check | Medium | CWE-639 | pattern | Beta |
| PF-GRAPHQL-DOS-001 | GraphQL DoS: missing depth/complexity limit configuration | Medium | CWE-400 | pattern | Experimental |

## Sources

The pack treats the following as untrusted user input inside resolvers:

- `info.context`
- `context.request`
- `context.args`
- `info.variable_values`
- `info.field_asts`
- `kwargs`

## Sinks

| Function | Arg Index |
| --- | --- |
| `text` | 0 |
| `execute` | 0 |
| `session.execute` | 0 |
| `db.session.execute` | 0 |
| `requests.get` | 0 |
| `requests.post` | 0 |
| `httpx.get` | 0 |
| `httpx.post` | 0 |
| `open` | 0 |
| `send_file` | 0 |
| `send_from_directory` | 0 |

## Sanitizers

The following functions and mechanisms suppress findings when present:

- `bindparam`
- `execute` with params dict
- `url_has_allowed_host_and_scheme`
- `is_safe_url`
- `secure_filename`
- `safe_join`
- `escape`
- `markupsafe.escape`
- `html.escape`

## Safe Patterns

- `:\w+` named parameter — suppresses **PF-GRAPHQL-SQLI-001** (parameterized SQL)
- `bindparam` — suppresses **PF-GRAPHQL-SQLI-001**
- `current_user` / `owner` / `user_id` / `auth` / `permission` / `authorize` —
  suppresses **PF-GRAPHQL-AUTH-001** (ownership check present)
- `depth_limit` / `complexity` / `cost_analysis` / `DepthLimit` / `CostAnalysis` /
  `validation_rules` — suppresses **PF-GRAPHQL-DOS-001** (depth/complexity limit configured)

## Exclusions

The pack excludes the following paths to reduce false positives from framework
internals and test fixtures:

- `django/**`
- `flask/**`
- `graphene/**`
- `ariadne/**`
- `tests/**`
- `docs/**`

## Usage

### Auto-detection

GraphQL is auto-detected when a `graphql-core`, `graphene`, `ariadne`, or
`strawberry-graphql` dependency is present.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework graphql
```

### Disable

```bash
patchflow scan run --disable-framework graphql
```

### List rules

```bash
patchflow rules list --framework graphql
```

### Explain a rule

```bash
patchflow explain --rule PF-GRAPHQL-SQLI-001
```

## Example Findings

### PF-GRAPHQL-SQLI-001 — SQL Injection

```python
import graphene
from sqlalchemy import text

class Query(graphene.ObjectType):
    user = graphene.Field(UserType, name=graphene.String())

    def resolve_user(self, info, name):
        # Vulnerable: resolver arg concatenated into raw SQL
        query = text("SELECT * FROM users WHERE name = '%s'" % name)
        return info.context.session.execute(query).fetchone()
```

**Expected finding:**

```
PF-GRAPHQL-SQLI-001  High  CWE-89
  schema.py:12
  GraphQL SQLi: resolver args flow to raw SQL (text/execute)
  Source: name (resolver arg)
  Sink:   session.execute(text(...))
```

**Fixed version (named parameters):**

```python
def resolve_user(self, info, name):
    query = text("SELECT * FROM users WHERE name = :name")
    return info.context.session.execute(query, {"name": name}).fetchone()
```

### PF-GRAPHQL-SSRF-001 — Server-Side Request Forgery

```python
import requests

class Query(graphene.ObjectType):
    preview = graphene.Field(StringType, url=graphene.String())

    def resolve_preview(self, info, url):
        # Vulnerable: outbound HTTP request to user-supplied URL
        resp = requests.get(url)
        return resp.text
```

**Expected finding:**

```
PF-GRAPHQL-SSRF-001  High  CWE-918
  schema.py:9
  GraphQL SSRF: resolver args flow to outbound HTTP request
  Source: url (resolver arg)
  Sink:   requests.get(url)
```

### PF-GRAPHQL-PATH-001 — Path Traversal

```python
from flask import send_file

class Query(graphene.ObjectType):
    file = graphene.Field(StringType, filename=graphene.String())

    def resolve_file(self, info, filename):
        # Vulnerable: user input flows to file path
        return send_file(f"/uploads/{filename}")
```

**Expected finding:**

```
PF-GRAPHQL-PATH-001  High  CWE-22
  schema.py:9
  GraphQL path traversal: resolver args flow to file operations
  Source: filename (resolver arg)
  Sink:   send_file(...)
```

### PF-GRAPHQL-AUTH-001 — IDOR / Missing Ownership Check

```python
class Query(graphene.ObjectType):
    document = graphene.Field(DocumentType, id=graphene.ID())

    def resolve_document(self, info, id):
        # Vulnerable: fetches by id without checking ownership
        return Document.query.get(id)
```

**Expected finding:**

```
PF-GRAPHQL-AUTH-001  Medium  CWE-639
  schema.py:8
  GraphQL authorization: resolver fetches object by id without ownership check
```

**Fixed version (ownership check):**

```python
def resolve_document(self, info, id):
    doc = Document.query.get(id)
    if doc is None or doc.user_id != info.context.current_user.id:
        return None
    return doc
```

### PF-GRAPHQL-DOS-001 — Missing Depth/Complexity Limit

```python
from ariadne import make_executable_schema

schema = make_executable_schema(type_defs, resolvers)
# Vulnerable: no depth_limit / cost_analysis validation rules configured
```

**Expected finding:**

```
PF-GRAPHQL-DOS-001  Medium  CWE-400
  schema.py:5
  GraphQL DoS: missing depth/complexity limit configuration
```

**Fixed version:**

```python
from ariadne import make_executable_schema
from graphql_depth_limit import depth_limit

schema = make_executable_schema(type_defs, resolvers)
validation_rules = [depth_limit(max_depth=10)]
```

## Custom Extensions

You can add organization-specific sources, sinks, sanitizers, and safe patterns
to the GraphQL pack. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for details.

```yaml
# .patchflow/framework-extensions.yml
framework: graphql
extensions:
  sources:
    - "info.context.user_input"
  sinks:
    - function: "custom_fetch"
      argIndex: 0
  sanitizers:
    - "validate_graphql_input"
```
