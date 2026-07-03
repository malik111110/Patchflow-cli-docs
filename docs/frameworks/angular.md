---
title: Angular Pack
description: 5 rules for TypeScript Angular — XSS via bypassSecurityTrust, template XSS, and open redirect
---

# Angular Pack

## Overview

The Angular pack provides framework-aware static analysis for Angular
applications. It understands Angular-specific sources (`route.queryParams`,
`route.params`, `ActivatedRoute.snapshot`, `FormControl.value`, `@Input`,
`ElementRef.nativeElement`, `HttpClient`), sinks (`bypassSecurityTrustHtml`,
`innerHTML`, `navigateByUrl`, `navigate`, `createComponent`), and sanitizers
(`DomSanitizer.sanitize`, `DOMPurify.sanitize`, `encodeURIComponent`,
`validateUrl`), and it scans Angular templates for `innerHTML` bindings.

| Property | Value |
| --- | --- |
| Pack name | `angular` |
| Language | TypeScript |
| File extensions | `.ts` |
| Template extensions | `.html` (Angular templates) |
| Rule count | 5 |
| Key coverage | XSS (bypassSecurityTrust), template XSS, open redirect |

Vulnerability categories covered:

- **CWE-79** Cross-Site Scripting (XSS) — 3 rules
- **CWE-601** Open Redirect — 2 rules

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-ANGULAR-XSS-001 | Angular XSS: `bypassSecurityTrustHtml` with route data | High | CWE-79 | pattern | Beta |
| PF-ANGULAR-XSS-002 | Angular template XSS: `innerHTML` binding | Medium | CWE-79 | template | Beta |
| PF-ANGULAR-XSS-003 | Angular XSS: route/form data flows to `bypassSecurityTrust*` or `innerHTML` | High | CWE-79 | taint | Experimental |
| PF-ANGULAR-REDIRECT-001 | Angular open redirect: router navigation with route input | Medium | CWE-601 | pattern | Beta |
| PF-ANGULAR-REDIRECT-002 | Angular open redirect: route data flows to router navigation | Medium | CWE-601 | taint | Experimental |

The pack combines pattern-based, template-based, and taint-based rules. The
taint rules (XSS-003, REDIRECT-002) track data flow from route/form sources
through component logic to sinks, catching cases the simpler pattern rules
miss.

## Sources

PatchFlow treats the following Angular inputs as taint sources:

- `route.queryParams`
- `route.params`
- `route.paramMap`
- `route.snapshot.paramMap`
- `route.snapshot.params`
- `route.snapshot.queryParams`
- `route.snapshot.url`
- `route.data`
- `route.fragment`
- `ActivatedRoute.queryParams`
- `ActivatedRoute.params`
- `ActivatedRoute.paramMap`
- `ActivatedRoute.snapshot`
- `queryParams`
- `paramMap`
- `location.search`
- `FormControl.value`
- `FormGroup.value`
- `formControl.value`
- `formGroup.value`
- `.value`
- `HttpClient`
- `http.get`
- `http.post`
- `@Input`
- `ElementRef.nativeElement`
- `elementRef.nativeElement`

## Sinks

| Function | Arg Index |
| --- | --- |
| `bypassSecurityTrustHtml` | 0 |
| `bypassSecurityTrustUrl` | 0 |
| `bypassSecurityTrustResourceUrl` | 0 |
| `bypassSecurityTrustScript` | 0 |
| `innerHTML` | -1 |
| `nativeElement.innerHTML` | -1 |
| `insertAdjacentHTML` | -1 |
| `outerHTML` | -1 |
| `navigateByUrl` | 0 |
| `navigate` | 0 |
| `window.location` | -1 |
| `document.location` | -1 |
| `createComponent` | -1 |
| `ViewContainerRef.createComponent` | -1 |

An arg index of `-1` means any argument; `0` means the first positional
argument is the tainted one.

## Sanitizers

Findings are suppressed when one of the following sanitizers wraps the tainted
data before it reaches a sink:

- `DomSanitizer.sanitize`
- `sanitizer.sanitize`
- `DOMPurify.sanitize`
- `sanitizeHtml`
- `encodeURIComponent`
- `isSafeUrl`
- `validateUrl`

## Safe Patterns

| Safe Pattern | Suppresses Rule |
| --- | --- |
| `DOMPurify.sanitize` | PF-ANGULAR-XSS-001 |
| `sanitizer.sanitize` | PF-ANGULAR-XSS-001 |

Sanitizing user-controlled data with `DOMPurify.sanitize` or Angular's
`DomSanitizer.sanitize` before passing it to `bypassSecurityTrustHtml`
suppresses the XSS finding.

## Usage

### Auto-detection

PatchFlow auto-detects Angular projects via `angular.json` /
`package.json` (@angular/core dependency) and activates the `angular` pack
automatically.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework angular
```

### List Angular rules

```bash
patchflow rules list --framework angular
```

### Explain a specific rule

```bash
patchflow explain --rule PF-ANGULAR-XSS-001
```

## Example Findings

### PF-ANGULAR-XSS-001 — `bypassSecurityTrustHtml` with route data

```ts
// src/app/components/profile/profile.component.ts
import { Component } from "@angular/core";
import { DomSanitizer } from "@angular/platform-browser";
import { ActivatedRoute } from "@angular/router";

@Component({
  selector: "app-profile",
  templateUrl: "./profile.component.html",
})
export class ProfileComponent {
  bio: any;
  constructor(route: ActivatedRoute, sanitizer: DomSanitizer) {
    this.bio = sanitizer.bypassSecurityTrustHtml(route.snapshot.queryParams["bio"]); // VULNERABLE
  }
}
```

**Expected finding:**

```
PF-ANGULAR-XSS-001  High  CWE-79
  src/app/components/profile/profile.component.ts:13
  Angular XSS: bypassSecurityTrustHtml with route data
  Route data (route.snapshot.queryParams) reaches bypassSecurityTrustHtml without sanitization.
```

**Remediation** — sanitize first, or avoid bypassing Angular's built-in
sanitization:

```ts
import DOMPurify from "dompurify";

this.bio = sanitizer.bypassSecurityTrustHtml(
  DOMPurify.sanitize(route.snapshot.queryParams["bio"])
);
// or simply bind as text and let Angular sanitize automatically:
// this.bio = route.snapshot.queryParams["bio"];
```

### PF-ANGULAR-XSS-002 — Template `innerHTML` binding

```html
<!-- src/app/components/profile/profile.component.html -->
<div [innerHTML]="bio"></div> <!-- VULNERABLE -->
```

**Expected finding:**

```
PF-ANGULAR-XSS-002  Medium  CWE-79
  src/app/components/profile/profile.component.html:1
  Angular template XSS: innerHTML binding
  innerHTML bound to user-controlled value without sanitization.
```

**Remediation** — Angular sanitizes `[innerHTML]` by default, but if the
value comes from `bypassSecurityTrustHtml`, the sanitization is bypassed.
Avoid bypassing, or sanitize explicitly:

```html
<div [innerHTML]="sanitizedBio"></div>
```

```ts
this.sanitizedBio = this.sanitizer.sanitize(
  SecurityContext.HTML,
  this.bio
);
```

### PF-ANGULAR-REDIRECT-001 — Open redirect via `navigateByUrl`

```ts
// src/app/components/login/login.component.ts
import { Component } from "@angular/core";
import { ActivatedRoute, Router } from "@angular/router";

@Component({
  selector: "app-login",
  templateUrl: "./login.component.html",
})
export class LoginComponent {
  constructor(route: ActivatedRoute, router: Router) {
    const returnTo = route.snapshot.queryParams["return_to"];
    this.router.navigateByUrl(returnTo); // VULNERABLE
  }
}
```

**Remediation** — validate the URL against an allowlist:

```ts
const ALLOWED = ["/dashboard", "/profile"];
const returnTo = route.snapshot.queryParams["return_to"];
if (ALLOWED.includes(returnTo)) {
  this.router.navigateByUrl(returnTo);
}
```

### PF-ANGULAR-XSS-003 — Taint: form data → `innerHTML`

```ts
// src/app/components/comment/comment.component.ts
import { Component, ElementRef, ViewChild } from "@angular/core";
import { FormControl } from "@angular/forms";

@Component({
  selector: "app-comment",
  template: `<input [formControl]="body" /><div #out></div>`,
})
export class CommentComponent {
  body = new FormControl("");
  @ViewChild("out") out!: ElementRef;

  ngDoCheck() {
    this.out.nativeElement.innerHTML = this.body.value; // VULNERABLE (taint)
  }
}
```

**Expected finding:**

```
PF-ANGULAR-XSS-003  High  CWE-79
  src/app/components/comment/comment.component.ts:13
  Angular XSS: route/form data flows to bypassSecurityTrust* or innerHTML
  FormControl.value (source) flows to nativeElement.innerHTML (sink) without sanitization.
```

**Remediation** — sanitize with DOMPurify (a safe pattern):

```ts
this.out.nativeElement.innerHTML = DOMPurify.sanitize(this.body.value);
```

## Custom Extensions

You can extend the Angular pack with organization-specific sources, sinks,
sanitizers, and safe patterns. For example, to register a custom sanitizer
used by your internal helpers:

```yaml
# .patchflow/framework-extensions.yml
pack: angular
extensions:
  sanitizers:
    - AppSecuritySanitizer.clean
  safe_patterns:
    - pattern: "DOMPurify.sanitize"
      suppresses: PF-ANGULAR-XSS-001
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full extension schema and advanced examples.
