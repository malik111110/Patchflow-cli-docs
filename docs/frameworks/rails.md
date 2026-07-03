---
title: Rails Pack
description: 15 rules for Ruby Rails — XSS, SQLi, open redirect, file disclosure, deserialization, mass assignment, and auth bypass
---

# Rails Pack

## Overview

The Rails pack provides framework-aware static analysis for Ruby on Rails
applications. It understands Rails-specific sources (`params`, `cookies`,
`request.headers`), sinks (`raw`, `html_safe`, `find_by_sql`, `redirect_to`),
and sanitizers (`sanitize`, `h`, `sanitize_sql`), and it scans ERB templates
for unescaped output.

| Property | Value |
| --- | --- |
| Pack name | `rails` |
| Language | Ruby |
| File extensions | `.rb` |
| Template extensions | `.erb` |
| Rule count | 15 |
| Key coverage | XSS, SQL injection, open redirect, file disclosure, unsafe deserialization, mass assignment, auth bypass |

Vulnerability categories covered:

- **CWE-79** Cross-Site Scripting (XSS) — 3 rules
- **CWE-89** SQL Injection — 3 rules
- **CWE-601** Open Redirect — 2 rules
- **CWE-73** External Control of File Name or Path — 1 rule
- **CWE-502** Unsafe Deserialization — 1 rule
- **CWE-915** Mass Assignment — 1 rule
- **CWE-306** Missing Authentication — 1 rule

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-RAILS-XSS-001 | Rails XSS: unescaped output via `raw`/`html_safe` | High | CWE-79 | pattern | Experimental |
| PF-RAILS-XSS-002 | Rails XSS: `render html:` with user input | High | CWE-79 | pattern | Experimental |
| PF-RAILS-XSS-003 | Rails template XSS: `raw`/`html_safe` in ERB | High | CWE-79 | template | Experimental |
| PF-RAILS-SQLI-001 | Rails SQLi: `find_by_sql` with interpolated string | High | CWE-89 | pattern | Experimental |
| PF-RAILS-SQLI-002 | Rails SQLi: `where` with string interpolation | High | CWE-89 | pattern | Experimental |
| PF-RAILS-SQLI-003 | Rails SQLi (taint): `params` → `find_by_sql`/`where` string | High | CWE-89 | taint | Experimental |
| PF-RAILS-REDIRECT-001 | Rails open redirect: `redirect_to` with user input | Medium | CWE-601 | pattern | Experimental |
| PF-RAILS-REDIRECT-002 | Rails open redirect (taint): `params` → `redirect_to` | Medium | CWE-601 | taint | Experimental |
| PF-RAILS-FILE-001 | Rails file disclosure: `send_file` with user input | High | CWE-73 | pattern | Experimental |
| PF-RAILS-DESER-001 | Rails unsafe deserialization: `YAML.load` / `Marshal.load` | High | CWE-502 | pattern | Experimental |
| PF-RAILS-MASS-001 | Rails mass assignment: `permit!` allows all attributes | Medium | CWE-915 | pattern | Experimental |
| PF-RAILS-AUTH-001 | Rails auth bypass: `skip_before_action :authenticate_user!` | Medium | CWE-306 | pattern | Experimental |

## Sources

PatchFlow treats the following Rails request inputs as taint sources:

- `params`
- `cookies`
- `request.headers`
- `request.query_parameters`
- `request.request_parameters`
- `request.GET`
- `request.POST`

## Sinks

| Function | Arg Index |
| --- | --- |
| `raw` | -1 |
| `html_safe` | -1 |
| `redirect_to` | 0 |
| `send_file` | 0 |
| `send_data` | -1 |
| `find_by_sql` | 0 |
| `constantize` | -1 |
| `public_send` | -1 |
| `render` | -1 |
| `system` | 0 |
| `exec` | 0 |
| `spawn` | 0 |
| `Net::HTTP.get` | -1 |
| `Net::HTTP.post` | -1 |
| `Net::HTTP.start` | -1 |
| `URI.parse` | 0 |
| `URI.open` | 0 |
| `Digest::MD5` | -1 |
| `Digest::SHA1` | -1 |

An arg index of `-1` means any argument; `0` means the first positional
argument is the tainted one.

## Sanitizers

Findings are suppressed when one of the following sanitizers wraps the tainted
data before it reaches a sink:

- `sanitize`
- `html_escape`
- `ERB::Util.html_escape`
- `ERB::Util.h`
- `h(`
- `.permit(([^*!)]+))` — strong parameters with an explicit allowlist
- `where(`
- `find_by(`
- `sanitize_sql`
- `sanitize_sql_array`
- `sanitize_sql_for`
- `url_for`
- `Shellwords.escape`
- `Shellwords.shellescape`

## Safe Patterns

| Safe Pattern | Suppresses Rule |
| --- | --- |
| `YAML.safe_load` | PF-RAILS-DESER-001 |

## Usage

### Auto-detection

PatchFlow auto-detects Rails projects via `Gemfile` / `config/application.rb`
and activates the `rails` pack automatically.

```bash
patchflow scan run
```

### Force-enable

```bash
patchflow scan run --framework rails
```

### Combine with other packs

```bash
patchflow scan run --framework rails --framework react
```

### List Rails rules

```bash
patchflow rules list --framework rails
```

### Explain a specific rule

```bash
patchflow explain --rule PF-RAILS-SQLI-001
```

## Example Findings

### PF-RAILS-XSS-001 — Unescaped output via `html_safe`

```ruby
# app/controllers/posts_controller.rb
class PostsController < ApplicationController
  def show
    @post = Post.find(params[:id])
    @title = params[:title].html_safe # VULNERABLE
  end
end
```

```erb
<!-- app/views/posts/show.html.erb -->
<h1><%= @title %></h1>
```

**Expected finding:**

```
PF-RAILS-XSS-001  High  CWE-79
  app/controllers/posts_controller.rb:4
  Rails XSS: unescaped output via raw/html_safe
  User-controlled input (params) reaches html_safe without sanitization.
```

**Remediation** — use Rails' built-in escaping or `sanitize`:

```ruby
@title = ERB::Util.h(params[:title])
# or, if limited HTML is required:
@title = sanitize(params[:title])
```

### PF-RAILS-SQLI-001 — `find_by_sql` with interpolated string

```ruby
# app/controllers/search_controller.rb
class SearchController < ApplicationController
  def index
    @results = Post.find_by_sql("SELECT * FROM posts WHERE title LIKE '%#{params[:q]}%'")
  end
end
```

**Expected finding:**

```
PF-RAILS-SQLI-001  High  CWE-89
  app/controllers/search_controller.rb:3
  Rails SQLi: find_by_sql with interpolated string
  User input (params[:q]) interpolated directly into a SQL string.
```

**Remediation** — use parameterized queries:

```ruby
@results = Post.where("title LIKE ?", "%#{params[:q]}%")
```

### PF-RAILS-REDIRECT-001 — Open redirect via `redirect_to`

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  def create
    # ... authenticate ...
    redirect_to params[:return_to] # VULNERABLE
  end
end
```

**Expected finding:**

```
PF-RAILS-REDIRECT-001  Medium  CWE-601
  app/controllers/sessions_controller.rb:4
  Rails open redirect: redirect_to with user input
  redirect_to called with unvalidated user-controlled URL.
```

**Remediation** — validate against an allowlist:

```ruby
return_to = params[:return_to]
redirect_to return_to if return_to.start_with?("/") && !return_to.start_with?("//")
```

### PF-RAILS-DESER-001 — `YAML.load` with user input

```ruby
# app/controllers/imports_controller.rb
class ImportsController < ApplicationController
  def import
    data = YAML.load(params[:payload]) # VULNERABLE
    process(data)
  end
end
```

**Remediation** — use `YAML.safe_load` (a safe pattern that suppresses this
rule):

```ruby
data = YAML.safe_load(params[:payload], permitted_classes: [Symbol])
```

### PF-RAILS-MASS-001 — `permit!` allows all attributes

```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def update
    User.update(params.require(:user).permit!) # VULNERABLE
  end
end
```

**Remediation** — allowlist specific attributes:

```ruby
User.update(params.require(:user).permit(:name, :email))
```

### PF-RAILS-AUTH-001 — Skipped authentication

```ruby
# app/controllers/admin_controller.rb
class AdminController < ApplicationController
  skip_before_action :authenticate_user! # VULNERABLE
  def dashboard; end
end
```

**Remediation** — remove the skip or scope it to non-sensitive actions only:

```ruby
skip_before_action :authenticate_user!, only: [:public_health_check]
```

## Custom Extensions

You can extend the Rails pack with organization-specific sources, sinks,
sanitizers, and safe patterns. For example, to register a custom sanitizer
used by your internal helpers:

```yaml
# .patchflow/framework-extensions.yml
pack: rails
extensions:
  sanitizers:
    - MySanitizer.escape
  safe_patterns:
    - pattern: "MySerializer.safe_render"
      suppresses: PF-RAILS-XSS-001
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full extension schema and advanced examples.
