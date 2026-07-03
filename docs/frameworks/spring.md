---
title: Spring Pack
description: 31 rules for Java Spring Framework — SQLi, SSRF, open redirect, XXE, deserialization, XSS, command injection, path traversal, and auth bypass
---

# Spring Pack

## Overview

The Spring pack covers the core Spring Framework ecosystem, including
`JdbcTemplate`, `EntityManager`, `RestTemplate`, `WebClient`, Thymeleaf and JSP
templates, XML processing, and the servlet API.

| Property | Value |
| --- | --- |
| Pack name | `spring` |
| Language | Java |
| File extensions | `.java` |
| Template extensions | `.jsp`, `.jspx`, `.ftl`, `.vm`, `.html`, `.thymeleaf.html` |
| Rule count | 31 |
| Match modes | `pattern`, `template` |
| Maturity | `experimental` (audit profile only) |

### Vulnerability categories covered

SQL Injection (CWE-89), SSRF (CWE-918), Open Redirect (CWE-601), Unsafe
Deserialization (CWE-502), XXE (CWE-611), Missing Authentication (CWE-306),
XSS (CWE-79), Command Injection (CWE-78), Path Traversal (CWE-22).

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-SPRING-SQLI-001 | Spring SQLi: JdbcTemplate query with string concatenation | High | CWE-89 | pattern | Experimental |
| PF-SPRING-SQLI-002 | Spring SQLi: EntityManager createNativeQuery with concatenation | High | CWE-89 | pattern | Experimental |
| PF-SPRING-SQLI-003 | Spring SQLi: string concatenation in SQL query with request input | High | CWE-89 | pattern | Experimental |
| PF-SPRING-SSRF-001 | Spring SSRF: RestTemplate with user-controlled URL | High | CWE-918 | pattern | Experimental |
| PF-SPRING-SSRF-002 | Spring SSRF: WebClient with user-controlled URL | High | CWE-918 | pattern | Experimental |
| PF-SPRING-SSRF-003 | Spring SSRF: new URL with user-controlled input | Medium | CWE-918 | pattern | Experimental |
| PF-SPRING-REDIRECT-001 | Spring open redirect: sendRedirect with user input | Medium | CWE-601 | pattern | Experimental |
| PF-SPRING-REDIRECT-002 | Spring open redirect: RedirectView with user input | Medium | CWE-601 | pattern | Experimental |
| PF-SPRING-REDIRECT-003 | Spring open redirect: ResponseEntity Location header with user input | Medium | CWE-601 | pattern | Experimental |
| PF-SPRING-DESER-001 | Spring unsafe deserialization: ObjectInputStream.readObject | Critical | CWE-502 | pattern | Experimental |
| PF-SPRING-DESER-002 | Spring unsafe deserialization: XStream.fromXML | Critical | CWE-502 | pattern | Experimental |
| PF-SPRING-XXE-001 | Spring XXE: DocumentBuilderFactory without secure configuration | High | CWE-611 | pattern | Experimental |
| PF-SPRING-XXE-002 | Spring XXE: SAXParserFactory without secure configuration | High | CWE-611 | pattern | Experimental |
| PF-SPRING-AUTH-001 | Spring auth bypass: permitAll on sensitive route | High | CWE-306 | pattern | Experimental |
| PF-SPRING-AUTH-002 | Spring auth bypass: web.ignoring on application route | High | CWE-306 | pattern | Experimental |
| PF-SPRING-XSS-001 | Spring XSS: Thymeleaf th:utext with user input | High | CWE-79 | template | Experimental |
| PF-SPRING-XSS-002 | Spring XSS: JSP c:out escapeXml=false | High | CWE-79 | template | Experimental |
| PF-SPRING-CMDI-001 | Spring command injection: Runtime.exec with user input | Critical | CWE-78 | pattern | Experimental |
| PF-SPRING-CMDI-002 | Spring command injection: ProcessBuilder with user input | Critical | CWE-78 | pattern | Experimental |
| PF-SPRING-PATH-001 | Spring path traversal: File/Paths with user input | High | CWE-22 | pattern | Experimental |

## Sources

Untrusted data entry points recognized by the Spring pack:

- `@RequestParam`
- `@PathVariable`
- `@RequestBody`
- `@RequestHeader`
- `@CookieValue`
- `@ModelAttribute`
- `getParameter`
- `getParameterValues`
- `getHeader`
- `getHeaders`
- `getQueryString`
- `getCookies`
- `getRequestURI`
- `getPathInfo`
- `ServerHttpRequest`

## Sinks

| Function | Arg Index |
| --- | --- |
| `jdbcTemplate.query` | 0 |
| `jdbcTemplate.queryForObject` | 0 |
| `jdbcTemplate.queryForList` | 0 |
| `jdbcTemplate.queryForMap` | 0 |
| `jdbcTemplate.update` | 0 |
| `jdbcTemplate.execute` | 0 |
| `entityManager.createNativeQuery` | 0 |
| `createNativeQuery` | 0 |
| `restTemplate.getForObject` | 0 |
| `restTemplate.postForObject` | 0 |
| `restTemplate.exchange` | 0 |
| `webClient.get` | -1 |
| `webClient.post` | -1 |
| `RestTemplate` | -1 |
| `WebClient` | -1 |
| `URL.openConnection` | 0 |
| `new URL` | 0 |
| `ObjectInputStream.readObject` | -1 |
| `readObject` | -1 |
| `XStream.fromXML` | 0 |
| `fromXML` | 0 |
| `DocumentBuilderFactory.newDocumentBuilder` | -1 |
| `DocumentBuilder.parse` | 0 |
| `SAXParserFactory.newSAXParser` | -1 |
| `SAXParser.parse` | 0 |
| `XMLInputFactory.createXMLStreamReader` | 0 |
| `response.sendRedirect` | 0 |
| `sendRedirect` | 0 |
| `RedirectView` | 0 |
| `ResponseEntity.Location` | -1 |
| `Runtime.getRuntime` | -1 |
| `Runtime.exec` | 0 |
| `ProcessBuilder` | -1 |
| `ProcessBuilder.start` | -1 |
| `new File` | 0 |
| `Paths.get` | 0 |
| `FileInputStream` | 0 |
| `Files.readAllBytes` | 0 |
| `Files.newInputStream` | 0 |

## Sanitizers

The following functions and patterns suppress findings when present in the
scanned code path:

- `NamedParameterJdbcTemplate`
- `PreparedStatement`
- `prepareStatement`
- `setParameter`
- `bind`
- `NamedQuery`
- `setFeature`
- `disallow-doctype-decl`
- `external-general-entities.*false`
- `external-parameter-entities.*false`
- `XMLConstants.FEATURE_SECURE_PROCESSING`
- `FEATURE_SECURE_PROCESSING`
- `setExpandEntityReferences(false)`
- `ExternalEntityResolver`
- `UriComponentsBuilder`
- `ServletUriComponentsBuilder`
- `isLocalUrl`
- `ObjectInputFilter`
- `resolveClass`
- `XStream.*allowTypes`
- `XStream.*allowTypeHierarchy`
- `HtmlUtils.htmlEscape`
- `StringEscapeUtils.escapeHtml`
- `escapeHtml4`
- `escapeHtml3`
- `OWASP HtmlSanitizer`
- `HtmlSanitizer`
- `CommandUtils`
- `ProcessBuilder with List.of/Arrays.asList`

## Safe Patterns

| Pattern | Suppresses |
| --- | --- |
| `ObjectInputFilter` | PF-SPRING-DESER-001 |
| `setFeature.*true` | PF-SPRING-XXE-001, PF-SPRING-XXE-002 |
| `hasRole\|hasAuthority\|authenticated` | PF-SPRING-AUTH-001, PF-SPRING-AUTH-002 |

## Usage

### Auto-detection

PatchFlow auto-detects Spring when it finds Spring Boot config files,
`pom.xml`/`build.gradle` with Spring dependencies, or `@RestController` /
`@Controller` annotations.

```bash
patchflow scan run
```

### Force-enable the Spring pack

```bash
patchflow scan run --framework spring
```

### Combine with Spring Security

```bash
patchflow scan run --framework spring --framework spring-security
```

### List Spring rules

```bash
patchflow rules list --framework spring
```

### Explain a specific rule

```bash
patchflow explain --rule PF-SPRING-SQLI-001
```

## Example Findings

### PF-SPRING-SQLI-001 — JdbcTemplate query with string concatenation

```java
@GetMapping("/orders")
public List<Order> orders(@RequestParam String status) {
    String sql = "SELECT * FROM orders WHERE status = '" + status + "'";
    return jdbcTemplate.query(sql, new OrderRowMapper());
}
```

```text
PF-SPRING-SQLI-001  High  CWE-89
  Spring SQLi: JdbcTemplate query with string concatenation
  src/main/java/com/example/OrderController.java:14
  Source: @RequestParam  Sink: jdbcTemplate.query
  Fix: use NamedParameterJdbcTemplate with :status named parameter
```

### PF-SPRING-XXE-001 — DocumentBuilderFactory without secure configuration

```java
@PostMapping(value = "/import", consumes = "application/xml")
public void importXml(@RequestBody String xml) throws Exception {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    // No setFeature calls — vulnerable to XXE
    DocumentBuilder builder = factory.newDocumentBuilder();
    Document doc = builder.parse(new ByteArrayInputStream(xml.getBytes()));
}
```

### PF-SPRING-XSS-001 — Thymeleaf th:utext with user input

```html
<p th:utext="${userComment}">placeholder</p>
```

`th:utext` renders unescaped HTML. Use `th:text` (which escapes by default) or
sanitize with `HtmlSanitizer` before rendering.

### PF-SPRING-AUTH-001 — permitAll on sensitive route

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/admin/**").permitAll()  // sensitive route, no auth
        .anyRequest().authenticated());
    return http.build();
}
```

## Custom Extensions

Extend the Spring pack with organization-specific sources, sinks, sanitizers,
and safe patterns in `.patchflow/rules.yaml`:

```yaml
schema_version: "1.0"

framework_extensions:
  spring:
    custom_sources:
      - annotation: "@TenantInput"
        categories: [sql_injection, path_traversal]
    custom_sinks:
      - function: "LegacySql.run"
        cwe: "CWE-89"
        category: "sql_injection"
    custom_sanitizers:
      - function: "CompanySql.safe"
    safe_patterns:
      - pattern: "TenantAuth.requireOwner"
        reason: "Ownership validation by internal auth helper"
```

Validate before committing:

```bash
patchflow rules validate .patchflow/rules.yaml
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full schema reference.
