---
title: Express Pack
description: 9 rules for JavaScript Express — SQLi, open redirect, XSS, command injection, path traversal, NoSQL injection, and SSRF
---

# Express Pack

## Overview

The Express pack covers the Express.js request/response model, including
`req.query`, `req.params`, `req.body`, template engines (EJS, Handlebars, Pug),
and common data-access libraries (`knex`, `sequelize`, generic `db.query`).

| Property | Value |
| --- | --- |
| Pack name | `express` |
| Language | JavaScript |
| File extensions | `.js`, `.mjs`, `.cjs`, `.ts` |
| Template extensions | `.ejs`, `.hbs`, `.pug` |
| Rule count | 9 |
| Match modes | `pattern` |
| Maturity | `beta` (dev, pr, audit profiles) |

### Vulnerability categories covered

SQL Injection (CWE-89), Open Redirect (CWE-601), XSS (CWE-79), Command
Injection (CWE-78), Path Traversal (CWE-22), NoSQL Injection (CWE-943), SSRF
(CWE-918).

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-EXPRESS-SQLI-001 | Express SQLi: query built from request data | High | CWE-89 | pattern | Beta |
| PF-EXPRESS-REDIRECT-001 | Express open redirect: res.redirect with request input | Medium | CWE-601 | pattern | Beta |
| PF-EXPRESS-XSS-001 | Express XSS: response sends request data as HTML | High | CWE-79 | pattern | Beta |
| PF-EXPRESS-CMDI-001 | Express command injection: child_process with request data | Critical | CWE-78 | pattern | Beta |
| PF-EXPRESS-PATH-001 | Express path traversal: filesystem access with request data | High | CWE-22 | pattern | Beta |
| PF-EXPRESS-SQLI-002 | Express SQLi: raw SQL query built from request data via knex/sequelize/db | High | CWE-89 | pattern | Beta |
| PF-EXPRESS-NOSQL-001 | Express NoSQL injection: raw request object passed to MongoDB query | Medium | CWE-943 | pattern | Beta |
| PF-EXPRESS-SSRF-001 | Express SSRF: HTTP client request URL built from request data | High | CWE-918 | pattern | Beta |
| PF-EXPRESS-XSS-002 | Express XSS: res.send/res.render with unsanitized request data | Medium | CWE-79 | pattern | Beta |

## Sources

- `req.query`
- `req.params`
- `req.body`
- `req.headers`
- `req.cookies`

## Sinks

| Function | Arg Index |
| --- | --- |
| `query` | 0 |
| `knex.raw` | 0 |
| `sequelize.query` | 0 |
| `db.query` | 0 |
| `res.redirect` | 0 |
| `res.send` | 0 |
| `res.render` | 1 |
| `child_process.exec` | 0 |
| `exec` | 0 |
| `execSync` | 0 |
| `spawn` | 0 |
| `axios.get` | 0 |
| `fetch` | 0 |
| `fs.readFile` | 0 |
| `fs.createReadStream` | 0 |

## Sanitizers

- `escapeHtml`
- `validator.escape`
- `DOMPurify.sanitize`
- `express-validator`
- `encodeURIComponent`
- `isSafeRedirect`
- `allowlistedHost`
- `path.resolve`

## Safe Patterns

| Pattern | Suppresses |
| --- | --- |
| `res.render` with string-literal template name | PF-EXPRESS-XSS-002 |

## Usage

### Auto-detection

PatchFlow auto-detects Express from `package.json` with `express` as a
dependency, or `app = express()` / `require('express')` references.

```bash
patchflow scan run
```

### Force-enable the Express pack

```bash
patchflow scan run --framework express
```

### Combine with React/Next.js in a full-stack project

```bash
patchflow scan run --framework express --framework react --framework nextjs
```

### List rules

```bash
patchflow rules list --framework express
```

### Explain a rule

```bash
patchflow explain --rule PF-EXPRESS-SQLI-001
```

## Example Findings

### PF-EXPRESS-SQLI-001 — query built from request data

```javascript
app.get('/users', (req, res) => {
  const sql = `SELECT * FROM users WHERE email = '${req.query.email}'`;
  db.query(sql, (err, rows) => {
    res.json(rows);
  });
});
```

```text
PF-EXPRESS-SQLI-001  High  CWE-89
  Express SQLi: query built from request data
  src/routes/users.js:12
  Source: req.query  Sink: db.query
  Fix: use parameterized query — db.query('SELECT * FROM users WHERE email = ?', [req.query.email])
```

### PF-EXPRESS-REDIRECT-001 — res.redirect with request input

```javascript
app.get('/login/callback', (req, res) => {
  const next = req.query.next;
  res.redirect(next);  // open redirect
});
```

Validate `next` against an allowlist or use `isSafeRedirect(next)` before
redirecting.

### PF-EXPRESS-NOSQL-001 — raw request object passed to MongoDB query

```javascript
app.get('/products', async (req, res) => {
  const results = await Product.find({ category: req.query });
  res.json(results);
});
```

Passing the entire `req.query` object enables NoSQL operators like
`{$ne: null}`. Extract and validate specific fields instead:
`{ category: req.query.category }`.

### PF-EXPRESS-CMDI-001 — child_process with request data

```javascript
import { exec } from 'child_process';

app.post('/convert', (req, res) => {
  exec(`convert ${req.body.filename} out.png`, (err, stdout) => {
    res.send(stdout);
  });
});
```

Use `spawn` with an argument array (`spawn('convert', [req.body.filename,
'out.png'])`) and validate the filename against an allowlist.

## Custom Extensions

Extend the Express pack with organization-specific sources, sinks, sanitizers,
and safe patterns:

```yaml
schema_version: "1.0"

framework_extensions:
  express:
    custom_sources:
      - function: "getTenantParam"
    custom_sinks:
      - function: "legacyDb.run"
        cwe: "CWE-89"
        category: "sql_injection"
    custom_sanitizers:
      - function: "validateRedirectTarget"
      - regex: "allowlistedHost\\("
    safe_patterns:
      - pattern: "assertTenantScope"
        reason: "Tenant scope validation before data access"
```

```bash
patchflow rules validate .patchflow/rules.yaml
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full schema reference.
