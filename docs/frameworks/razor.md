---
title: Razor Pack
description: 2 rules for C# Razor — XSS via Html.Raw and MarkupString with user-controlled data
---

# Razor Pack

## Overview

The Razor pack detects Cross-Site Scripting (XSS) vulnerabilities in Razor
template files (`.cshtml` and `.razor`). Razor auto-encodes output by default,
but `Html.Raw` and `MarkupString` bypass encoding, making them dangerous when
used with user-controlled data.

- **Pack name:** `razor`
- **Language:** C#
- **File extensions:** `.cshtml`, `.razor`
- **Template extensions:** `.cshtml`, `.razor`
- **Rules:** 2
- **Key vulnerability categories:** Cross-Site Scripting (XSS)

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-RAZOR-XSS-001 | Razor XSS: Html.Raw with user-controlled data | High | CWE-79 | template | Beta |
| PF-RAZOR-XSS-002 | Razor XSS: MarkupString with user-controlled data | High | CWE-79 | template | Beta |

## Sources

The pack treats the following as untrusted user input in Razor templates:

- `Request.Query`
- `Model`
- `ViewBag`
- `ViewData`

## Sinks

| Function | Arg Index |
| --- | --- |
| `Html.Raw` | 0 |
| `MarkupString` | 0 |

## Sanitizers

The following functions suppress findings when present:

- `Html.Encode`
- `WebUtility.HtmlEncode`
- `HtmlEncoder.Default.Encode`

## Safe Patterns

- Using `Html.Encode`, `WebUtility.HtmlEncode`, or `HtmlEncoder.Default.Encode`
  before passing data to `Html.Raw` or `MarkupString` suppresses both rules.
- Standard Razor output (`@Model.Property`) auto-encodes and does not trigger
  findings.

## Usage

### Auto-detection

Razor is auto-detected when `.cshtml` or `.razor` files are present in the
project.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework razor
```

### Disable

```bash
patchflow scan run --disable-framework razor
```

### List rules

```bash
patchflow rules list --framework razor
```

### Explain a rule

```bash
patchflow explain --rule PF-RAZOR-XSS-001
```

## Example Findings

### PF-RAZOR-XSS-001 — Html.Raw with User Input

```cshtml
@{
    var userName = Request.Query["name"];
}

<!-- Vulnerable: user input rendered without encoding -->
<div>@Html.Raw(userName)</div>
```

**Expected finding:**

```
PF-RAZOR-XSS-001  High  CWE-79
  Index.cshtml:6
  Razor XSS: Html.Raw with user-controlled data
  Source: Request.Query["name"]
  Sink:   Html.Raw(userName)
```

**Fixed version (let Razor auto-encode):**

```cshtml
@{
    var userName = Request.Query["name"];
}

<div>@userName</div>
```

**Fixed version (explicit encoding if raw output is required):**

```cshtml
@{
    var userName = Request.Query["name"];
    var encoded = HtmlEncoder.Default.Encode(userName);
}

<div>@Html.Raw(encoded)</div>
```

### PF-RAZOR-XSS-002 — MarkupString with User Input

```razor
@code {
    private string userInput = "";
}

@* Vulnerable: user input in MarkupString bypasses encoding *@
<div>@((MarkupString)userInput)</div>
```

**Expected finding:**

```
PF-RAZOR-XSS-002  High  CWE-79
  Component.razor:7
  Razor XSS: MarkupString with user-controlled data
  Source: userInput
  Sink:   (MarkupString)userInput
```

**Fixed version:**

```razor
@code {
    private string userInput = "";
}

<div>@userInput</div>
```

## Custom Extensions

You can add organization-specific sources, sinks, sanitizers, and safe patterns
to the Razor pack. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for details.

```yaml
# .patchflow/framework-extensions.yml
framework: razor
extensions:
  sources:
    - "TempData"
  sinks:
    - function: "CustomRaw"
      argIndex: 0
  sanitizers:
    - "MyHtmlSanitizer"
```
