---
title: Spring Security Pack
description: 4 rules for Java Spring Security — CSRF disabled, permitAll on sensitive routes, web.ignoring, and @PermitAll
---

# Spring Security Pack

## Overview

The Spring Security pack focuses on the security configuration layer of Spring
applications: `HttpSecurity`, `SecurityFilterChain`, `WebSecurity`, and JSR-250
authorization annotations. It detects misconfigurations that disable CSRF or
expose sensitive routes without authentication.

| Property | Value |
| --- | --- |
| Pack name | `spring-security` |
| Language | Java |
| File extensions | `.java` |
| Template extensions | — |
| Rule count | 4 |
| Match modes | `pattern` |
| Maturity | `beta` (dev, pr, audit profiles) |

### Vulnerability categories covered

CSRF (CWE-352), Missing Authentication (CWE-306).

## Rules

| ID | Title | Severity | CWE | Match Mode | Maturity |
| --- | --- | --- | --- | --- | --- |
| PF-SPRINGSEC-CSRF-001 | Spring Security CSRF disabled | Medium | CWE-352 | pattern | Beta |
| PF-SPRINGSEC-AUTH-001 | Spring Security auth bypass: permitAll on sensitive route | High | CWE-306 | pattern | Beta |
| PF-SPRINGSEC-AUTH-002 | Spring Security bypass: web.ignoring on application route | High | CWE-306 | pattern | Beta |
| PF-SPRINGSEC-AUTH-003 | Spring Security auth bypass: @PermitAll annotation | Medium | CWE-306 | pattern | Beta |

## Sources

- `HttpSecurity`
- `SecurityFilterChain`
- `@RequestMapping`

## Sinks

| Function | Arg Index |
| --- | --- |
| `permitAll` | -1 |
| `csrf.disable` | -1 |
| `web.ignoring` | -1 |
| `@PermitAll` | -1 |

## Sanitizers

- `hasRole`
- `hasAuthority`
- `authenticated`
- `@PreAuthorize`

## Safe Patterns

The sanitizer list above doubles as the safe-pattern set for this pack. When
`hasRole`, `hasAuthority`, `authenticated`, or `@PreAuthorize` appears on a
route, the corresponding auth-bypass finding is suppressed.

## Usage

### Auto-detection

PatchFlow auto-detects Spring Security when it finds
`spring-security-config` dependencies or `SecurityFilterChain` / `WebSecurity`
references.

```bash
patchflow scan run
```

### Force-enable the pack

```bash
patchflow scan run --framework spring-security
```

### Combine with the core Spring pack

```bash
patchflow scan run --framework spring --framework spring-security
```

### Disable the pack

```bash
patchflow scan run --disable-framework spring-security
```

`--disable-framework` takes precedence over `--framework`.

### List rules

```bash
patchflow rules list --framework spring-security
```

### Explain a rule

```bash
patchflow explain --rule PF-SPRINGSEC-AUTH-001
```

## Example Findings

### PF-SPRINGSEC-CSRF-001 — CSRF disabled

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf(csrf -> csrf.disable());
    http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated());
    return http.build();
}
```

```text
PF-SPRINGSEC-CSRF-001  Medium  CWE-352
  Spring Security CSRF disabled
  src/main/java/com/example/SecurityConfig.java:12
  Sink: csrf.disable
  Fix: keep CSRF enabled for state-changing endpoints; disable only for stateless APIs
```

### PF-SPRINGSEC-AUTH-001 — permitAll on sensitive route

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/admin/**").permitAll()
        .anyRequest().authenticated());
    return http.build();
}
```

```text
PF-SPRINGSEC-AUTH-001  High  CWE-306
  Spring Security auth bypass: permitAll on sensitive route
  src/main/java/com/example/SecurityConfig.java:15
  Sink: permitAll
  Fix: replace permitAll with hasRole('ADMIN') or hasAuthority('admin:read')
```

### PF-SPRINGSEC-AUTH-002 — web.ignoring on application route

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return web -> web.ignoring().requestMatchers("/api/internal/**");
}
```

`web.ignoring` skips the entire security filter chain (including CSRF and
authentication) for the matched routes. Use `authorizeHttpRequests` with
`permitAll` instead if the route still needs CSRF protection.

### PF-SPRINGSEC-AUTH-003 — @PermitAll annotation

```java
@RestController
@RequestMapping("/api/admin")
public class AdminController {

    @PermitAll
    @GetMapping("/users")
    public List<User> listUsers() {
        return userService.findAll();
    }
}
```

## Custom Extensions

Extend the Spring Security pack with organization-specific sanitizers and safe
patterns:

```yaml
schema_version: "1.0"

framework_extensions:
  spring-security:
    custom_sanitizers:
      - function: "TenantAuth.requireRole"
    safe_patterns:
      - pattern: "InternalAuth.assertAdmin"
        reason: "Internal admin authorization guard"
```

```bash
patchflow rules validate .patchflow/rules.yaml
```

See the [Custom Framework Extensions guide](../user-guides/custom-framework-extensions)
for the full schema reference.
