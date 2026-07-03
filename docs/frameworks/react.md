---
title: React Pack
description: 4 rules for JavaScript React — XSS via dangerouslySetInnerHTML, open redirect, DOM injection, and insecure storage
---

# React Pack

## Overview

The React pack provides framework-aware static analysis for React
applications. It understands React-specific sources (`props`, `state`,
`location.search`, `useSearchParams`, `useParams`), sinks
(`dangerouslySetInnerHTML`, `window.location`, `innerHTML`,
`localStorage.setItem`), and sanitizers (`DOMPurify.sanitize`,
`sanitizeHtml`, `encodeURIComponent`), and it scans JSX for
`dangerouslySetInnerHTML` usage.

| Property | Value |
| --- | --- |
| Pack name | `react` |
| Language | JavaScript |
| File extensions | `.jsx`, `.tsx` |
| Template extensions | `.jsx`, `.tsx` (JSX) |
| Rule count | 4 |
| Key coverage | XSS (dangerouslySetInnerHTML), open redirect, DOM injection, insecure storage |

Vulnerability categories covered:

- **CWE-79** Cross-Site Scripting (XSS) — 2 rules
- **CWE-601** Open Redirect — 1 rule
- **CWE-922** Insecure Storage — 1 rule

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-REACT-XSS-001 | React XSS: `dangerouslySetInnerHTML` with user-controlled data | High | CWE-79 | template | Beta |
| PF-REACT-REDIRECT-001 | React open redirect: navigation with user-controlled URL | Medium | CWE-601 | pattern | Beta |
| PF-REACT-XSS-002 | React DOM injection: `ref.current.innerHTML` / `insertAdjacentHTML` with user-controlled data | High | CWE-79 | pattern | Beta |
| PF-REACT-STORAGE-001 | React insecure storage: token-like secrets stored in `localStorage`/`sessionStorage` | Medium | CWE-922 | pattern | Beta |

## Sources

PatchFlow treats the following React inputs as taint sources:

- `props`
- `state`
- `location.search`
- `URLSearchParams`
- `useSearchParams`
- `useParams`
- `useLocation`
- `response`
- `data`

## Sinks

| Function | Arg Index |
| --- | --- |
| `dangerouslySetInnerHTML` | -1 |
| `window.location` | -1 |
| `router.push` | 0 |
| `innerHTML` | -1 |
| `insertAdjacentHTML` | -1 |
| `localStorage.setItem` | -1 |
| `sessionStorage.setItem` | -1 |

An arg index of `-1` means any argument; `0` means the first positional
argument is the tainted one.

## Sanitizers

Findings are suppressed when one of the following sanitizers wraps the tainted
data before it reaches a sink:

- `DOMPurify.sanitize`
- `sanitizeHtml`
- `encodeURIComponent`
- `isSafeRedirect`
- `textContent`

## Safe Patterns

| Safe Pattern | Suppresses Rule |
| --- | --- |
| `DOMPurify.sanitize` | PF-REACT-XSS-002 |

Wrapping user-controlled data with `DOMPurify.sanitize` before assigning it to
`innerHTML` suppresses the DOM-injection finding.

## Usage

### Auto-detection

PatchFlow auto-detects React projects via `package.json` (react dependency)
and JSX/TSX files, and activates the `react` pack automatically.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework react
```

### Combine with Next.js pack

```bash
patchflow scan run --framework react --framework nextjs
```

### List React rules

```bash
patchflow rules list --framework react
```

### Explain a specific rule

```bash
patchflow explain --rule PF-REACT-XSS-001
```

## Example Findings

### PF-REACT-XSS-001 — `dangerouslySetInnerHTML` with user-controlled data

```jsx
// src/components/Comment.jsx
export default function Comment({ comment }) {
  return (
    <div
      dangerouslySetInnerHTML={{ __html: comment.body }} // VULNERABLE
    />
  );
}
```

**Expected finding:**

```
PF-REACT-XSS-001  High  CWE-79
  src/components/Comment.jsx:4
  React XSS: dangerouslySetInnerHTML with user-controlled data
  User-controlled prop (comment) reaches dangerouslySetInnerHTML without sanitization.
```

**Remediation** — sanitize with DOMPurify, or render as text:

```jsx
import DOMPurify from "dompurify";

export default function Comment({ comment }) {
  return (
    <div
      dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(comment.body) }}
    />
  );
}
// or simply render as text (React escapes automatically):
// <div>{comment.body}</div>
```

### PF-REACT-XSS-002 — DOM injection via `ref.current.innerHTML`

```jsx
// src/components/Preview.jsx
import { useRef, useEffect } from "react";

export default function Preview({ html }) {
  const ref = useRef(null);
  useEffect(() => {
    ref.current.innerHTML = html; // VULNERABLE
  }, [html]);
  return <div ref={ref} />;
}
```

**Remediation** — sanitize before assignment (a safe pattern that suppresses
this rule):

```jsx
ref.current.innerHTML = DOMPurify.sanitize(html);
```

### PF-REACT-REDIRECT-001 — Open redirect via `window.location`

```jsx
// src/components/Login.jsx
import { useSearchParams } from "react-router-dom";

export default function Login() {
  const [searchParams] = useSearchParams();
  const returnTo = searchParams.get("return_to");
  const handleLogin = () => {
    window.location = returnTo; // VULNERABLE
  };
  return <button onClick={handleLogin}>Login</button>;
}
```

**Remediation** — validate the URL against an allowlist:

```jsx
const ALLOWED = ["/dashboard", "/profile"];
const handleLogin = () => {
  if (ALLOWED.includes(returnTo)) {
    window.location = returnTo;
  }
};
```

### PF-REACT-STORAGE-001 — Token stored in `localStorage`

```jsx
// src/components/Auth.jsx
export default function Auth() {
  const handleLogin = async () => {
    const res = await fetch("/api/login", { method: "POST" });
    const { token } = await res.json();
    localStorage.setItem("auth_token", token); // VULNERABLE
  };
  return <button onClick={handleLogin}>Login</button>;
}
```

**Remediation** — store tokens in an HttpOnly cookie instead of
`localStorage`, which is accessible to any JavaScript running on the page
(including XSS payloads):

```js
// Set the token as an HttpOnly cookie on the server side.
// Do not persist tokens in localStorage or sessionStorage.
```

## Custom Extensions

You can extend the React pack with organization-specific sources, sinks,
sanitizers, and safe patterns. For example, to register a custom sanitizer
used by your internal helpers:

```yaml
# .patchflow/framework-extensions.yml
pack: react
extensions:
  sanitizers:
    - mySanitizer.clean
  safe_patterns:
    - pattern: "DOMPurify.sanitize"
      suppresses: PF-REACT-XSS-002
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full extension schema and advanced examples.
