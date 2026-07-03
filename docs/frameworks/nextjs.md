---
title: Next.js Pack
description: 4 rules for JavaScript Next.js — SSRF, open redirect, XSS, and secret exposure to client components
---

# Next.js Pack

## Overview

The Next.js pack provides framework-aware static analysis for Next.js
applications. It understands Next.js-specific sources (`req.query`,
`searchParams`, `params`, `cookies`, `headers`, `NextRequest`,
`request.nextUrl`), sinks (`fetch`, `redirect`, `NextResponse.redirect`,
`process.env.NEXT_PUBLIC_`), and sanitizers (`encodeURIComponent`,
`isSafeRedirect`, `new URL`, `server-only`), and it scans JSX for
`dangerouslySetInnerHTML` usage.

| Property | Value |
| --- | --- |
| Pack name | `nextjs` |
| Language | JavaScript |
| File extensions | `.js`, `.jsx`, `.ts`, `.tsx` |
| Template extensions | `.jsx`, `.tsx` (JSX) |
| Rule count | 4 |
| Key coverage | SSRF, open redirect, XSS, secret exposure to client components |

Vulnerability categories covered:

- **CWE-918** Server-Side Request Forgery (SSRF) — 1 rule
- **CWE-601** Open Redirect — 1 rule
- **CWE-79** Cross-Site Scripting (XSS) — 1 rule
- **CWE-200** Information Exposure — 1 rule

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-NEXTJS-SSRF-001 | Next.js SSRF: `fetch` with request-controlled URL | High | CWE-918 | pattern | Beta |
| PF-NEXTJS-REDIRECT-001 | Next.js open redirect: `redirect` with request input | Medium | CWE-601 | pattern | Beta |
| PF-NEXTJS-XSS-001 | Next.js XSS: `dangerouslySetInnerHTML` with request data | High | CWE-79 | template | Beta |
| PF-NEXTJS-SECRET-001 | Next.js secret exposed to client component or `NEXT_PUBLIC_` misuse | Medium | CWE-200 | pattern | Beta |

## Sources

PatchFlow treats the following Next.js request inputs as taint sources:

- `req.query`
- `searchParams`
- `params`
- `cookies`
- `headers`
- `request.nextUrl.searchParams`
- `request.nextUrl`
- `NextRequest`
- `formData`

## Sinks

| Function | Arg Index |
| --- | --- |
| `fetch` | 0 |
| `redirect` | 0 |
| `NextResponse.redirect` | 0 |
| `router.push` | 0 |
| `axios.get` | 0 |
| `process.env.NEXT_PUBLIC_` | -1 |

An arg index of `-1` means any argument; `0` means the first positional
argument is the tainted one.

## Sanitizers

Findings are suppressed when one of the following sanitizers wraps the tainted
data before it reaches a sink:

- `encodeURIComponent`
- `isSafeRedirect`
- `allowlistedHost`
- `new URL`
- `server-only`

Importing `server-only` marks a module as server-only, preventing it (and any
secrets it references) from being bundled into client components.

## Safe Patterns

This pack does not currently define explicit safe-pattern suppressions beyond
the sanitizer list above. Using `new URL` to validate a redirect target, or
`isSafeRedirect` / `allowlistedHost` helpers, suppresses the open-redirect
finding; importing `server-only` suppresses the secret-exposure finding for
that module.

## Usage

### Auto-detection

PatchFlow auto-detects Next.js projects via `next.config.js` /
`next.config.mjs` / `package.json` (next dependency) and activates the
`nextjs` pack automatically.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework nextjs
```

### Combine with React pack

```bash
patchflow scan run --framework nextjs --framework react
```

### List Next.js rules

```bash
patchflow rules list --framework nextjs
```

### Explain a specific rule

```bash
patchflow explain --rule PF-NEXTJS-SSRF-001
```

## Example Findings

### PF-NEXTJS-SSRF-001 — `fetch` with request-controlled URL

```js
// app/api/proxy/route.js
export async function GET(request) {
  const target = request.nextUrl.searchParams.get("url");
  const res = await fetch(target); // VULNERABLE
  return new Response(await res.text());
}
```

**Expected finding:**

```
PF-NEXTJS-SSRF-001  High  CWE-918
  app/api/proxy/route.js:3
  Next.js SSRF: fetch with request-controlled URL
  User input (request.nextUrl.searchParams) reaches fetch without validation.
```

**Remediation** — validate the URL against an allowlist of hosts:

```js
import "server-only";

const ALLOWED_HOSTS = ["api.example.com", "cdn.example.com"];

export async function GET(request) {
  const target = request.nextUrl.searchParams.get("url");
  const url = new URL(target);
  if (!ALLOWED_HOSTS.includes(url.hostname)) {
    return new Response("Disallowed host", { status: 400 });
  }
  const res = await fetch(url);
  return new Response(await res.text());
}
```

### PF-NEXTJS-REDIRECT-001 — Open redirect via `redirect`

```js
// app/login/actions.js
import { redirect } from "next/navigation";

export async function login(formData) {
  // ... authenticate ...
  redirect(formData.get("return_to")); // VULNERABLE
}
```

**Remediation** — validate the redirect target:

```js
const returnTo = formData.get("return_to");
const ALLOWED = ["/dashboard", "/profile"];
if (ALLOWED.includes(returnTo)) {
  redirect(returnTo);
}
redirect("/dashboard");
```

### PF-NEXTJS-XSS-001 — `dangerouslySetInnerHTML` with request data

```jsx
// app/page.jsx
export default function Page({ searchParams }) {
  return (
    <div
      dangerouslySetInnerHTML={{ __html: searchParams.content }} // VULNERABLE
    />
  );
}
```

**Remediation** — sanitize with DOMPurify, or render as text:

```jsx
import DOMPurify from "dompurify";

export default function Page({ searchParams }) {
  return (
    <div
      dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(searchParams.content) }}
    />
  );
}
```

### PF-NEXTJS-SECRET-001 — Secret exposed to client component

```jsx
// app/components/ApiClient.jsx (client component — no "server-only")
"use client";

const API_KEY = process.env.NEXT_PUBLIC_API_KEY; // VULNERABLE
// NEXT_PUBLIC_ vars are inlined into the client bundle and visible to users.

export async function fetchData() {
  const res = await fetch("/api/data", { headers: { Authorization: `Bearer ${API_KEY}` } });
  return res.json();
}
```

**Remediation** — keep secrets server-side and proxy through a route handler:

```js
// app/api/data/route.js
import "server-only";

const API_KEY = process.env.API_KEY; // not NEXT_PUBLIC_

export async function GET() {
  const res = await fetch("https://api.example.com/data", {
    headers: { Authorization: `Bearer ${API_KEY}` },
  });
  return Response.json(await res.json());
}
```

## Custom Extensions

You can extend the Next.js pack with organization-specific sources, sinks,
sanitizers, and safe patterns. For example, to register a custom sanitizer
used by your internal helpers:

```yaml
# .patchflow/framework-extensions.yml
pack: nextjs
extensions:
  sanitizers:
    - mySanitizer.clean
  safe_patterns:
    - pattern: "isSafeRedirect("
      suppresses: PF-NEXTJS-REDIRECT-001
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full extension schema and advanced examples.
