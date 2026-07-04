---
title: Framework Packs
description: 18 official embedded framework rule packs for framework-aware SAST analysis
---

# Framework Packs

Framework packs are official embedded rule sets that understand framework-specific
sources, sinks, sanitizers, and template engines. They run alongside the general
SAST engines and provide framework-aware vulnerability detection.

## How It Works

```
1. Detect frameworks    Filesystem signals (config files, manifests, directories)
2. Select packs         Detection results + CLI flags (--framework / --disable-framework)
3. Match patterns       Line-oriented matching with sanitizer awareness
4. Track taint          Source → sink taint analysis through the taint engine
5. Check safe patterns  Suppress findings when safe patterns are present
6. Deduplicate          SAST runner deduplicates findings across all engines
```

## Enabling Packs

### Auto-detection (default)

```bash
patchflow scan run
```

PatchFlow auto-detects frameworks based on filesystem signals and activates the
appropriate packs automatically.

### Force-enable specific packs

```bash
patchflow scan run --framework express
patchflow scan run --framework express --framework react --framework nextjs
```

### Disable a pack

```bash
patchflow scan run --disable-framework spring-security
```

`--disable-framework` takes precedence over `--framework`.

### List all packs and detection status

```bash
patchflow rules list-frameworks
```

## All 18 Framework Packs

| Pack | Language | Rules | File Extensions | Key Coverage |
| --- | --- | --- | --- | --- |
| Spring | Java | 31 | `.java` | SQLi, SSRF, redirect, XXE, deserialization, XSS, command injection, path traversal, auth bypass |
| Spring Security | Java | 4 | `.java` | CSRF disabled, permitAll on sensitive routes, @PermitAll |
| Express | JavaScript | 9 | `.js`, `.mjs`, `.cjs`, `.ts` | SQLi, XSS, redirect, command injection, path traversal, NoSQL injection, SSRF |
| Django | Python | 7 | `.py` | SQLi, XSS, redirect, deserialization, CSRF, SSRF |
| Flask | Python | 9 | `.py` | SQLi, SSRF, redirect, XSS, SSTI, path traversal, debug mode |
| FastAPI | Python | 9 | `.py` | SQLi, SSRF, redirect, command injection, path traversal, XSS, auth |
| Rails | Ruby | 15 | `.rb` | XSS, SQLi, redirect, file disclosure, deserialization, mass assignment, auth bypass |
| Laravel | PHP | 6 | `.php` | SQLi, redirect, XSS, mass assignment, deserialization, auth |
| Symfony | PHP | 3 | `.php` | SQLi, redirect, Twig XSS |
| React | JavaScript | 4 | `.jsx`, `.tsx` | XSS (dangerouslySetInnerHTML), redirect, DOM injection, insecure storage |
| Next.js | JavaScript | 4 | `.js`, `.jsx`, `.ts`, `.tsx` | SSRF, redirect, XSS, secret exposure |
| Angular | TypeScript | 5 | `.ts` | XSS (bypassSecurityTrust), redirect, template XSS |
| NestJS | TypeScript | 4 | `.ts` | SQLi, SSRF, redirect, auth |
| GraphQL | Python | 5 | `.py` | SQLi, SSRF, path traversal, IDOR, DoS |
| ASP.NET | C# | 8 | `.cs` | SQLi, redirect, XSS, deserialization, command injection, path traversal |
| Razor | C# | 2 | `.cshtml`, `.razor` | XSS (Html.Raw, MarkupString) |
| Gin | Go | 4 | `.go` | SQLi, redirect, XSS, command injection |
| Echo | Go | 3 | `.go` | SQLi, redirect, XSS |

## Rule Details by Language

Expand a framework to see its full rule table, sources, sinks, and sanitizers.

### Java

<Accordion>
  <AccordionGroup title="Spring (31 rules)">
    Pack name: `spring` · File extensions: `.java` · Template extensions: `.jsp`, `.jspx`, `.ftl`, `.vm`, `.html`, `.thymeleaf.html` · Maturity: `experimental` (audit profile only)

    **Vulnerability categories:** SQLi (CWE-89), SSRF (CWE-918), Open Redirect (CWE-601), Deserialization (CWE-502), XXE (CWE-611), Missing Auth (CWE-306), XSS (CWE-79), Command Injection (CWE-78), Path Traversal (CWE-22)

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-SPRING-SQLI-001 | JdbcTemplate query with string concatenation | High | CWE-89 | pattern | Experimental |
    | PF-SPRING-SQLI-002 | EntityManager createNativeQuery with concatenation | High | CWE-89 | pattern | Experimental |
    | PF-SPRING-SQLI-003 | String concatenation in SQL query with request input | High | CWE-89 | pattern | Experimental |
    | PF-SPRING-SSRF-001 | RestTemplate with user-controlled URL | High | CWE-918 | pattern | Experimental |
    | PF-SPRING-SSRF-002 | WebClient with user-controlled URL | High | CWE-918 | pattern | Experimental |
    | PF-SPRING-SSRF-003 | new URL with user-controlled input | Medium | CWE-918 | pattern | Experimental |
    | PF-SPRING-REDIRECT-001 | sendRedirect with user input | Medium | CWE-601 | pattern | Experimental |
    | PF-SPRING-REDIRECT-002 | RedirectView with user input | Medium | CWE-601 | pattern | Experimental |
    | PF-SPRING-REDIRECT-003 | ResponseEntity Location header with user input | Medium | CWE-601 | pattern | Experimental |
    | PF-SPRING-DESER-001 | ObjectInputStream.readObject | Critical | CWE-502 | pattern | Experimental |
    | PF-SPRING-DESER-002 | XStream.fromXML | Critical | CWE-502 | pattern | Experimental |
    | PF-SPRING-XXE-001 | DocumentBuilderFactory without secure configuration | High | CWE-611 | pattern | Experimental |
    | PF-SPRING-XXE-002 | SAXParserFactory without secure configuration | High | CWE-611 | pattern | Experimental |
    | PF-SPRING-AUTH-001 | permitAll on sensitive route | High | CWE-306 | pattern | Experimental |
    | PF-SPRING-AUTH-002 | web.ignoring on application route | High | CWE-306 | pattern | Experimental |
    | PF-SPRING-XSS-001 | Thymeleaf th:utext with user input | High | CWE-79 | template | Experimental |
    | PF-SPRING-XSS-002 | JSP c:out escapeXml=false | High | CWE-79 | template | Experimental |
    | PF-SPRING-CMDI-001 | Runtime.exec with user input | Critical | CWE-78 | pattern | Experimental |
    | PF-SPRING-CMDI-002 | ProcessBuilder with user input | Critical | CWE-78 | pattern | Experimental |
    | PF-SPRING-PATH-001 | File/Paths with user input | High | CWE-22 | pattern | Experimental |

    **Sources:** `@RequestParam`, `@PathVariable`, `@RequestBody`, `@RequestHeader`, `@CookieValue`, `HttpServletRequest.getParameter`, `HttpServletRequest.getHeader`, `request.getParameter`, `request.getHeader`, `Model.addAttribute`

    **Sinks:** `JdbcTemplate.query`, `EntityManager.createNativeQuery`, `RestTemplate`, `WebClient`, `sendRedirect`, `RedirectView`, `ObjectInputStream.readObject`, `XStream.fromXML`, `DocumentBuilderFactory`, `SAXParserFactory`, `Runtime.exec`, `ProcessBuilder`, `File`, `Paths.get`

    **Sanitizers:** `escapeHtml`, `escapeXml`, `URLEncoder.encode`, `ResponseEntity.noContent`, `XStream.allowTypes`, `setFeature("http://apache.org/xml/features/disallow-doctype-decl"...)`, `ParameterizedPreparedStatementSetter`, `new SqlParameterSource`, `Paths.normalize`
  </AccordionGroup>

  <AccordionGroup title="Spring Security (4 rules)">
    Pack name: `spring-security` · File extensions: `.java` · Maturity: `experimental`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-SPRINGSEC-CSRF-001 | CSRF protection disabled | High | CWE-352 | pattern | Experimental |
    | PF-SPRINGSEC-AUTH-001 | permitAll on sensitive route | High | CWE-306 | pattern | Experimental |
    | PF-SPRINGSEC-AUTH-002 | web.ignoring on application route | High | CWE-306 | pattern | Experimental |
    | PF-SPRINGSEC-AUTH-003 | @PermitAll on sensitive method | High | CWE-306 | pattern | Experimental |

    **Sources:** `@PreAuthorize`, `@PostAuthorize`, `@Secured`, `@RolesAllowed`, `SecurityConfig`, `WebSecurityConfigurerAdapter`

    **Sinks:** `csrf().disable()`, `permitAll()`, `web.ignoring()`, `@PermitAll`

    **Sanitizers:** `@PreAuthorize`, `@PostAuthorize`, `@Secured`, `@DenyAll`, `authenticated()`, `hasRole()`, `hasAuthority()`
  </AccordionGroup>
</Accordion>

### JavaScript / TypeScript

<Accordion>
  <AccordionGroup title="Express (9 rules)">
    Pack name: `express` · File extensions: `.js`, `.mjs`, `.cjs`, `.ts` · Template extensions: `.ejs`, `.hbs`, `.pug` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-EXPRESS-SQLI-001 | query built from request data | High | CWE-89 | pattern | Beta |
    | PF-EXPRESS-REDIRECT-001 | res.redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-EXPRESS-XSS-001 | response sends request data as HTML | High | CWE-79 | pattern | Beta |
    | PF-EXPRESS-CMDI-001 | child_process with request data | Critical | CWE-78 | pattern | Beta |
    | PF-EXPRESS-PATH-001 | filesystem access with request data | High | CWE-22 | pattern | Beta |
    | PF-EXPRESS-SQLI-002 | raw SQL query via knex/sequelize/db | High | CWE-89 | pattern | Beta |
    | PF-EXPRESS-NOSQL-001 | raw request object passed to MongoDB query | Medium | CWE-943 | pattern | Beta |
    | PF-EXPRESS-SSRF-001 | HTTP client request URL built from request data | High | CWE-918 | pattern | Beta |
    | PF-EXPRESS-XSS-002 | res.send/res.render with unsanitized request data | Medium | CWE-79 | pattern | Beta |

    **Sources:** `req.query`, `req.params`, `req.body`, `req.headers`, `req.cookies`

    **Sinks:** `query`, `knex.raw`, `sequelize.query`, `db.query`, `res.redirect`, `res.send`, `res.render`, `child_process.exec`, `exec`, `execSync`, `spawn`, `axios.get`, `fetch`, `fs.readFile`, `fs.createReadStream`

    **Sanitizers:** `escapeHtml`, `validator.escape`, `DOMPurify.sanitize`, `express-validator`, `encodeURIComponent`, `isSafeRedirect`, `allowlistedHost`, `path.resolve`
  </AccordionGroup>

  <AccordionGroup title="React (4 rules)">
    Pack name: `react` · File extensions: `.jsx`, `.tsx` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-REACT-XSS-001 | dangerouslySetInnerHTML with user input | High | CWE-79 | pattern | Beta |
    | PF-REACT-REDIRECT-001 | window.location with user input | Medium | CWE-601 | pattern | Beta |
    | PF-REACT-DOM-001 | innerHTML with user input | High | CWE-79 | pattern | Beta |
    | PF-REACT-STORAGE-001 | sensitive data in localStorage | Medium | CWE-922 | pattern | Beta |

    **Sources:** `props`, `useSearchParams`, `useLocation`, `useParams`, `URLSearchParams`, `window.location`, `localStorage`, `sessionStorage`

    **Sinks:** `dangerouslySetInnerHTML`, `innerHTML`, `insertAdjacentHTML`, `window.location`, `document.location`, `localStorage.setItem`, `sessionStorage.setItem`

    **Sanitizers:** `DOMPurify.sanitize`, `escapeHtml`, `encodeURIComponent`, `isSafeUrl`
  </AccordionGroup>

  <AccordionGroup title="Next.js (4 rules)">
    Pack name: `nextjs` · File extensions: `.js`, `.jsx`, `.ts`, `.tsx` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-NEXTJS-SSRF-001 | Server-side fetch with user-controlled URL | High | CWE-918 | pattern | Beta |
    | PF-NEXTJS-REDIRECT-001 | redirect() with user input | Medium | CWE-601 | pattern | Beta |
    | PF-NEXTJS-XSS-001 | dangerouslySetInnerHTML with user input | High | CWE-79 | pattern | Beta |
    | PF-NEXTJS-SECRET-001 | Secret exposed in client component | Medium | CWE-200 | pattern | Beta |

    **Sources:** `searchParams`, `params`, `useSearchParams`, `useRouter`, `req.query`, `req.body`, `req.headers`, `cookies()`, `headers()`

    **Sinks:** `fetch`, `axios`, `redirect`, `dangerouslySetInnerHTML`, `innerHTML`, `process.env` (in client components)

    **Sanitizers:** `DOMPurify.sanitize`, `encodeURIComponent`, `isSafeUrl`, `Server` directive (marks server component)
  </AccordionGroup>

  <AccordionGroup title="Angular (5 rules)">
    Pack name: `angular` · File extensions: `.ts` · Template extensions: `.html` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-ANGULAR-XSS-001 | bypassSecurityTrustHtml with route data | High | CWE-79 | pattern | Beta |
    | PF-ANGULAR-XSS-002 | innerHTML binding | Medium | CWE-79 | template | Beta |
    | PF-ANGULAR-XSS-003 | route/form data flows to bypassSecurityTrust*/innerHTML | High | CWE-79 | taint | Experimental |
    | PF-ANGULAR-REDIRECT-001 | router navigation with route input | Medium | CWE-601 | pattern | Beta |
    | PF-ANGULAR-REDIRECT-002 | route data flows to router navigation | Medium | CWE-601 | taint | Experimental |

    **Sources:** `route.queryParams`, `route.params`, `route.paramMap`, `route.snapshot.*`, `ActivatedRoute.*`, `queryParams`, `paramMap`, `location.search`, `FormControl.value`, `FormGroup.value`, `.value`, `@Input`, `ElementRef.nativeElement`

    **Sinks:** `bypassSecurityTrustHtml`, `bypassSecurityTrustUrl`, `bypassSecurityTrustResourceUrl`, `bypassSecurityTrustScript`, `innerHTML`, `nativeElement.innerHTML`, `insertAdjacentHTML`, `navigateByUrl`, `navigate`, `window.location`, `document.location`

    **Sanitizers:** `DomSanitizer.sanitize`, `sanitizer.sanitize`, `DOMPurify.sanitize`, `sanitizeHtml`, `encodeURIComponent`, `isSafeUrl`, `validateUrl`
  </AccordionGroup>

  <AccordionGroup title="NestJS (4 rules)">
    Pack name: `nestjs` · File extensions: `.ts` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-NESTJS-SQLI-001 | Raw SQL query with request data | High | CWE-89 | pattern | Beta |
    | PF-NESTJS-SSRF-001 | Outbound HTTP with user-controlled URL | High | CWE-918 | pattern | Beta |
    | PF-NESTJS-REDIRECT-001 | res.redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-NESTJS-AUTH-001 | Controller without @UseGuards or @Public | Medium | CWE-306 | pattern | Beta |

    **Sources:** `@Query`, `@Param`, `@Body`, `@Headers`, `@Req`, `req.query`, `req.params`, `req.body`, `req.headers`

    **Sinks:** `query`, `execute`, `createQueryBuilder.raw`, `axios.get`, `fetch`, `res.redirect`

    **Sanitizers:** `escapeHtml`, `encodeURIComponent`, `isSafeUrl`, `@UseGuards`, `@Public`
  </AccordionGroup>
</Accordion>

### Python

<Accordion>
  <AccordionGroup title="Django (7 rules)">
    Pack name: `django` · File extensions: `.py` · Template extensions: `.html`, `.jinja`, `.jinja2` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-DJANGO-SQLI-001 | raw SQL built from request data | High | CWE-89 | pattern | Beta |
    | PF-DJANGO-REDIRECT-001 | redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-DJANGO-XSS-001 | mark_safe with request data | High | CWE-79 | pattern | Beta |
    | PF-DJANGO-XSS-002 | template safe filter | High | CWE-79 | template | Beta |
    | PF-DJANGO-DESER-001 | pickle.loads on request data | Critical | CWE-502 | pattern | Beta |
    | PF-DJANGO-CSRF-001 | @csrf_exempt on view | Medium | CWE-352 | pattern | Beta |
    | PF-DJANGO-SSRF-001 | requests.get/httpx.get with user-controlled URL | High | CWE-918 | pattern | Beta |

    **Sources:** `request.GET`, `request.POST`, `request.COOKIES`, `request.headers`, `request.body`, `request.data`, `request.FILES`, `request.META`

    **Sinks:** `raw`, `extra`, `cursor.execute`, `RawSQL`, `redirect`, `mark_safe`, `pickle.loads`, `yaml.load`, `requests.get`, `httpx.get`, `subprocess.run`

    **Sanitizers:** `escape`, `conditional_escape`, `format_html`, `url_has_allowed_host_and_scheme`, `bleach.clean`, `yaml.safe_load`
  </AccordionGroup>

  <AccordionGroup title="Flask (9 rules)">
    Pack name: `flask` · File extensions: `.py` · Template extensions: `.html`, `.jinja`, `.jinja2` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-FLASK-SQLI-001 | execute with request data | High | CWE-89 | pattern | Beta |
    | PF-FLASK-SSRF-001 | outbound request with request-controlled URL | High | CWE-918 | pattern | Beta |
    | PF-FLASK-REDIRECT-001 | redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-FLASK-XSS-001 | Jinja safe filter | High | CWE-79 | template | Beta |
    | PF-FLASK-SSTI-001 | render_template_string with dynamic template | High | CWE-94 | pattern | Beta |
    | PF-FLASK-PATH-001 | send_file/open with request input | High | CWE-22 | pattern | Beta |
    | PF-FLASK-CONFIG-001 | debug mode enabled or hardcoded secret key | Medium | CWE-489 | pattern | Beta |
    | PF-FLASK-SQLI-002 | request data flows to raw SQL (taint) | High | CWE-89 | taint | Experimental |
    | PF-FLASK-SSRF-002 | request data flows to outbound HTTP (taint) | High | CWE-918 | taint | Experimental |

    **Sources:** `request.args`, `request.form`, `request.values`, `request.headers`, `request.cookies`, `request.json`, `request.get_json`, `request.files`, `request.data`

    **Sinks:** `cursor.execute`, `session.execute`, `db.session.execute`, `text`, `execute`, `requests.get`, `requests.post`, `httpx.get`, `httpx.post`, `redirect`, `Markup`, `render_template_string`, `send_file`, `send_from_directory`, `open`, `subprocess.run`, `os.system`

    **Sanitizers:** `escape`, `markupsafe.escape`, `html.escape`, `url_has_allowed_host_and_scheme`, `is_safe_url`, `secure_filename`
  </AccordionGroup>

  <AccordionGroup title="FastAPI (9 rules)">
    Pack name: `fastapi` · File extensions: `.py` · Template extensions: `.html`, `.jinja`, `.jinja2` · Maturity: `experimental`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-FASTAPI-SQLI-001 | execute with interpolated request data | High | CWE-89 | pattern | Experimental |
    | PF-FASTAPI-SSRF-001 | outbound HTTP call with request-controlled URL | High | CWE-918 | pattern | Experimental |
    | PF-FASTAPI-REDIRECT-001 | RedirectResponse with request input | Medium | CWE-601 | pattern | Experimental |
    | PF-FASTAPI-CMDI-001 | subprocess with request data | Critical | CWE-78 | pattern | Experimental |
    | PF-FASTAPI-PATH-001 | FileResponse/open with request input | High | CWE-22 | pattern | Experimental |
    | PF-FASTAPI-XSS-001 | Jinja safe filter | High | CWE-79 | template | Experimental |
    | PF-FASTAPI-SQLI-002 | request data reaches execute/text() (taint) | High | CWE-89 | taint | Experimental |
    | PF-FASTAPI-REDIRECT-002 | request data reaches RedirectResponse (taint) | Medium | CWE-601 | taint | Experimental |
    | PF-FASTAPI-AUTH-001 | sensitive endpoint without Depends(auth) | Medium | CWE-862 | pattern | Experimental |

    **Sources:** `request.query_params`, `request.path_params`, `request.headers`, `request.cookies`, `request.json`, `request.form`, `Query`, `Path`, `Header`, `Cookie`, `Body`

    **Sinks:** `execute`, `executemany`, `text`, `session.execute`, `requests.get`, `requests.post`, `httpx.get`, `httpx.post`, `RedirectResponse`, `subprocess.run`, `subprocess.Popen`, `subprocess.call`, `subprocess.check_output`, `FileResponse`, `open`

    **Sanitizers:** `html.escape`, `markupsafe.escape`, `urllib.parse.quote`, `is_safe_url`, `allow_redirect_host`, `Path.resolve`, `Path.is_relative_to`
  </AccordionGroup>

  <AccordionGroup title="GraphQL (5 rules)">
    Pack name: `graphql` · File extensions: `.py` · Maturity: `experimental`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-GRAPHQL-SQLI-001 | SQL query built from resolver argument | High | CWE-89 | pattern | Experimental |
    | PF-GRAPHQL-SSRF-001 | HTTP request with resolver-controlled URL | High | CWE-918 | pattern | Experimental |
    | PF-GRAPHQL-PATH-001 | File access with resolver argument | High | CWE-22 | pattern | Experimental |
    | PF-GRAPHQL-IDOR-001 | Object access without ownership check | Medium | CWE-639 | pattern | Experimental |
    | PF-GRAPHQL-DOS-001 | Query depth/complexity not limited | Medium | CWE-400 | pattern | Experimental |

    **Sources:** `info.variable_values`, `info.context`, `resolver args`, `GraphQLResolveInfo`, `context.request`

    **Sinks:** `execute`, `cursor.execute`, `session.execute`, `requests.get`, `httpx.get`, `open`, `FileResponse`

    **Sanitizers:** `escape`, `urllib.parse.quote`, `Path.resolve`, `get_queryset` (with ownership check), `depth_limit_validator`, `cost_analysis`
  </AccordionGroup>
</Accordion>

### Ruby

<Accordion>
  <AccordionGroup title="Rails (15 rules)">
    Pack name: `rails` · File extensions: `.rb` · Template extensions: `.erb`, `.html.erb` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-RAILS-XSS-001 | raw helper with user input | High | CWE-79 | pattern | Beta |
    | PF-RAILS-XSS-002 | html_safe with user input | High | CWE-79 | pattern | Beta |
    | PF-RAILS-XSS-003 | ERB template raw output | High | CWE-79 | template | Beta |
    | PF-RAILS-SQLI-001 | where with string interpolation | High | CWE-89 | pattern | Beta |
    | PF-RAILS-SQLI-002 | order/limit with string interpolation | High | CWE-89 | pattern | Beta |
    | PF-RAILS-SQLI-003 | execute/select_values with string | High | CWE-89 | pattern | Beta |
    | PF-RAILS-SQLI-004 | find_by_sql with string | High | CWE-89 | pattern | Beta |
    | PF-RAILS-REDIRECT-001 | redirect_to with request input | Medium | CWE-601 | pattern | Beta |
    | PF-RAILS-FILE-001 | send_file with user input | High | CWE-22 | pattern | Beta |
    | PF-RAILS-DESER-001 | Marshal.load with user input | Critical | CWE-502 | pattern | Beta |
    | PF-RAILS-DESER-002 | YAML.load with user input | Critical | CWE-502 | pattern | Beta |
    | PF-RAILS-MASS-001 | mass assignment without permit | Medium | CWE-915 | pattern | Beta |
    | PF-RAILS-AUTH-001 | skip_before_action :verify_user | High | CWE-306 | pattern | Beta |
    | PF-RAILS-AUTH-002 | before_action :authenticate_user, only (incomplete) | Medium | CWE-306 | pattern | Beta |
    | PF-RAILS-CMDI-001 | system/backticks with user input | Critical | CWE-78 | pattern | Beta |

    **Sources:** `params`, `request.headers`, `request.env`, `cookies`, `session`, `flash`

    **Sinks:** `raw`, `html_safe`, `where`, `order`, `limit`, `execute`, `select_values`, `find_by_sql`, `redirect_to`, `send_file`, `Marshal.load`, `YAML.load`, `system`, `exec`, backticks

    **Sanitizers:** `h`, `escapeHTML`, `ERB::Util.html_escape`, `CGI.escapeHTML`, `sanitize`, `permit`, `YAML.safe_load`, `Pathname`, `File.expand_path`
  </AccordionGroup>
</Accordion>

### PHP

<Accordion>
  <AccordionGroup title="Laravel (6 rules)">
    Pack name: `laravel` · File extensions: `.php` · Template extensions: `.blade.php` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-LARAVEL-SQLI-001 | DB::raw/whereRaw with user input | High | CWE-89 | pattern | Beta |
    | PF-LARAVEL-REDIRECT-001 | redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-LARAVEL-XSS-001 | Blade {!! !!} with user input | High | CWE-79 | template | Beta |
    | PF-LARAVEL-MASS-001 | mass assignment without $fillable | Medium | CWE-915 | pattern | Beta |
    | PF-LARAVEL-DESER-001 | unserialize with user input | Critical | CWE-502 | pattern | Beta |
    | PF-LARAVEL-AUTH-001 | route without auth middleware | Medium | CWE-306 | pattern | Beta |

    **Sources:** `$request->input`, `$request->query`, `$request->post`, `$request->all`, `$request->get`, `$_GET`, `$_POST`, `$_REQUEST`, `$_COOKIE`

    **Sinks:** `DB::raw`, `whereRaw`, `selectRaw`, `orderByRaw`, `redirect`, `{!! !!}`, `unserialize`, `exec`, `system`, `shell_exec`

    **Sanitizers:** `e()`, `htmlspecialchars`, `urlencode`, `$fillable`, `$guarded`, `serialize`, `auth middleware`
  </AccordionGroup>

  <AccordionGroup title="Symfony (3 rules)">
    Pack name: `symfony` · File extensions: `.php` · Template extensions: `.twig` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-SYMFONY-SQLI-001 | createQueryBuilder with string concatenation | High | CWE-89 | pattern | Beta |
    | PF-SYMFONY-REDIRECT-001 | redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-SYMFONY-XSS-001 | Twig raw filter with user input | High | CWE-79 | template | Beta |

    **Sources:** `$request->query`, `$request->request`, `$request->headers`, `$request->cookies`, `$request->get`

    **Sinks:** `createQueryBuilder`, `executeQuery`, `redirect`, `raw`, `render`

    **Sanitizers:** `escape`, `htmlspecialchars`, `urlencode`, `isSafeUrl`
  </AccordionGroup>
</Accordion>

### C\#

<Accordion>
  <AccordionGroup title="ASP.NET (8 rules)">
    Pack name: `aspnet` · File extensions: `.cs` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-ASPNET-SQLI-001 | raw SQL built from request data | High | CWE-89 | pattern | Beta |
    | PF-ASPNET-REDIRECT-001 | Redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-ASPNET-XSS-001 | raw HTML response from request data | High | CWE-79 | pattern | Beta |
    | PF-ASPNET-SQLI-002 | FromSqlRaw with string interpolation | High | CWE-89 | pattern | Beta |
    | PF-ASPNET-XSS-002 | @Html.Raw with user-controlled data | High | CWE-79 | pattern | Beta |
    | PF-ASPNET-DESER-001 | BinaryFormatter.Deserialize with user input | Critical | CWE-502 | pattern | Experimental |
    | PF-ASPNET-CMDI-001 | Process.Start with request data | Critical | CWE-78 | pattern | Experimental |
    | PF-ASPNET-PATH-001 | Path.Combine with user input | High | CWE-22 | pattern | Experimental |

    **Sources:** `Request.Query`, `Request.Form`, `Request.Headers`, `HttpContext.Request.Query`, `Request.RouteValues`, `RouteData.Values`, `[FromQuery]`, `[FromBody]`

    **Sinks:** `FromSqlRaw`, `ExecuteSqlRaw`, `SqlCommand`, `Redirect`, `HtmlString`, `Content`, `BinaryFormatter.Deserialize`, `Process.Start`, `Path.Combine`

    **Sanitizers:** `HtmlEncoder.Default.Encode`, `WebUtility.HtmlEncode`, `Url.IsLocalUrl`, `LocalRedirect`, `FromSqlInterpolated`, `ExecuteSqlInterpolated`, `new SqlParameter`, `Path.GetFullPath`, `JsonSerializer.Deserialize`
  </AccordionGroup>

  <AccordionGroup title="Razor (2 rules)">
    Pack name: `razor` · File extensions: `.cshtml`, `.razor` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-RAZOR-XSS-001 | Html.Raw with user input | High | CWE-79 | template | Beta |
    | PF-RAZOR-XSS-002 | MarkupString with user input | High | CWE-79 | template | Beta |

    **Sources:** `Model`, `ViewData`, `ViewBag`, `Request.Query`, `Request.Form`, `@Input`

    **Sinks:** `Html.Raw`, `MarkupString`, `innerHTML`

    **Sanitizers:** `HtmlEncoder.Default.Encode`, `WebUtility.HtmlEncode`, `@Html.DisplayFor`, `@Html.Encode`
  </AccordionGroup>
</Accordion>

### Go

<Accordion>
  <AccordionGroup title="Gin (4 rules)">
    Pack name: `gin` · File extensions: `.go` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-GIN-SQLI-001 | database query built from request data | High | CWE-89 | pattern | Beta |
    | PF-GIN-REDIRECT-001 | c.Redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-GIN-XSS-001 | c.HTML with request input | High | CWE-79 | pattern | Beta |
    | PF-GIN-CMDI-001 | exec.Command with request data | Critical | CWE-78 | pattern | Beta |

    **Sources:** `c.Query`, `c.Param`, `c.PostForm`, `c.GetHeader`, `c.Request.URL.Query`

    **Sinks:** `db.Raw`, `db.Exec`, `db.Query`, `c.Redirect`, `c.HTML`, `c.String`, `exec.Command`, `exec.CommandContext`

    **Sanitizers:** `html.EscapeString`, `url.QueryEscape`, `filepath.Clean`, `isSafeRedirect`, `template.HTML`
  </AccordionGroup>

  <AccordionGroup title="Echo (3 rules)">
    Pack name: `echo` · File extensions: `.go` · Maturity: `beta`

    | ID | Title | Severity | CWE | Mode | Maturity |
    | --- | --- | --- | --- | --- | --- |
    | PF-ECHO-SQLI-001 | database query built from request data | High | CWE-89 | pattern | Beta |
    | PF-ECHO-REDIRECT-001 | c.Redirect with request input | Medium | CWE-601 | pattern | Beta |
    | PF-ECHO-XSS-001 | c.HTML with request input | High | CWE-79 | pattern | Beta |

    **Sources:** `c.QueryParam`, `c.Param`, `c.FormValue`, `c.Request().Header.Get`

    **Sinks:** `db.Raw`, `db.Exec`, `c.Redirect`, `c.HTML`, `c.File`, `exec.Command`

    **Sanitizers:** `html.EscapeString`, `url.QueryEscape`, `filepath.Clean`, `isSafeRedirect`
  </AccordionGroup>
</Accordion>

## Rule Match Modes

Each rule uses one of four match modes:

| Mode | Description | Example |
| --- | --- | --- |
| `pattern` | Regex against source lines (simple dangerous APIs) | `JdbcTemplate.query` with string concatenation |
| `template` | Template engine output issues (ERB, Jinja, Blade, JSX) | `th:utext` with user input in Thymeleaf |
| `taint` | Source → sink taint tracking through the taint engine | `@RequestParam` → `jdbcTemplate.query` |
| `AST` | Framework-specific call structures (tree-sitter, reserved) | Reserved for future use |

## Rule Maturity

| Level | Meaning | Default in profiles |
| --- | --- | --- |
| `experimental` | New rule, limited test coverage | audit only |
| `beta` | Tested against fixtures, some regression coverage | dev, pr, audit |
| `stable` | Full test coverage, regression tested, low false-positive rate | all profiles |
| `enterprise` | Validated against enterprise codebases | all profiles |

## CWE Coverage

Framework packs cover 17 CWE categories:

| CWE | Category | Rules |
| --- | --- | --- |
| CWE-89 | SQL Injection | 22 |
| CWE-79 | Cross-Site Scripting (XSS) | 18 |
| CWE-601 | Open Redirect | 15 |
| CWE-918 | Server-Side Request Forgery (SSRF) | 10 |
| CWE-78 | Command Injection | 7 |
| CWE-502 | Unsafe Deserialization | 6 |
| CWE-22 | Path Traversal | 6 |
| CWE-306 | Missing Authentication | 5 |
| CWE-352 | CSRF | 3 |
| CWE-611 | XXE | 2 |
| CWE-639 | IDOR | 1 |
| CWE-915 | Mass Assignment | 2 |
| CWE-400 | DoS | 1 |
| CWE-94 | SSTI | 1 |
| CWE-489 | Debug Mode | 1 |
| CWE-200 | Secret Exposure | 1 |
| CWE-943 | NoSQL Injection | 1 |

## Explaining Rules

Get detailed information about any rule, including its sources, sinks,
sanitizers, and safe patterns:

```bash
patchflow explain --rule PF-SPRING-SQLI-001
```

## Listing Rules

List rules for a specific framework:

```bash
patchflow rules list --framework spring
```

List all rules across all scanners:

```bash
patchflow rules list --all
```

## Custom Framework Extensions

You can extend any of these 18 official packs with organization-specific
sources, sinks, sanitizers, and safe patterns. See the
[Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full schema reference and examples for each framework.
