---
title: "Spring Boot 4.0 Major Changes: 7 Migration Points Worth Checking First"
date: 2026-04-05 00:13:00 +0900
categories: [Backend, Springboot]
tags: [spring boot 4.0, spring boot 4 migration, spring framework 7, java 21, backend]
description: "A practical summary of the most important Spring Boot 4.0 changes and migration checkpoints."
image:
  path: /assets/img/posts/spring-boot-4-migration-cover.jpg
toc: true
---

> Korean version: [Spring Boot 4.0 변경점 정리: 실무에서 먼저 확인할 7가지 마이그레이션 포인트](/posts/spring-boot-4-0-important-changes/)

Spring Boot 4.0 feels less like a routine version update and more like a meaningful platform shift. The visible version bump is small, but the practical implications are not. Once you look at server choices, starter structure, configuration changes, testing, and observability, it becomes clear that this is a release that can change how a project is organized.

For teams already running Spring Boot 3.x in production, the better question is not just “what’s new?” but rather “what is likely to break, and what do we need to revisit?”

## Quick summary

The most important themes in Spring Boot 4.0 are:

- the baseline has moved up
- starter and module structure has become more explicit
- operations-oriented concerns such as observability matter even more

In other words, Spring Boot 4.0 is not just about new features. It is a version that pushes teams to re-check assumptions they may have carried for years.

## 1. Undertow support is gone

One of the most visible changes is the removal of Undertow support. Since Spring Boot 4.0 requires a Servlet 6.1 baseline and Undertow is not aligned with it, it is no longer available as an embedded server option.

That matters in practice because some teams have real production history, tuning habits, and operational playbooks built around Undertow. In such cases, upgrading is not just about changing a dependency version. It also means rethinking server behavior and revisiting operational defaults around Tomcat or Jetty.

### Checkpoints

- confirm whether the current application still depends on Undertow
- find Undertow-specific tuning left in configuration files
- re-check thread, timeout, and performance settings after moving away from it

## 2. Starter and module structure matters more now

Spring Boot 4.0 introduces a stronger modular design. Support is more clearly mapped to dedicated starters and modules, and that makes the dependency graph easier to reason about.

That is a good thing, but it also means that projects relying on “it worked because something else pulled it in” may need explicit cleanup.

Examples include:

- `spring-boot-starter-webmvc`
- `spring-boot-starter-webflux`
- `spring-boot-starter-restclient`
- `spring-boot-starter-webclient`
- `spring-boot-starter-opentelemetry`

For teams that previously added only third-party libraries without corresponding Spring Boot starters, this release is a good reason to review what is actually declared.

### Checkpoints

- identify technologies used without dedicated Spring Boot starters
- review Flyway, Liquibase, and other infra-oriented dependencies
- review test starters, especially for security and web layers

## 3. Configuration property changes can hurt more than code changes

Configuration is often where major upgrades become painful. In Spring Boot 4.0, some properties have been renamed or removed, and the impact may show up at runtime rather than during compilation.

That is why the official migration tooling matters. The `spring-boot-properties-migrator` can temporarily help detect renamed properties and smooth part of the transition.

It should be treated as a migration aid, not as a long-term dependency.

### Checkpoints

- review `application.yml` and `application.properties` carefully
- review environment variable names as well
- compare dev, test, and prod profile differences
- remove the migrator once migration is complete

## 4. HTTP client and API versioning support are more structured

Spring Boot 4.0 adds clearer support around HTTP service clients and API versioning. That makes it easier to revisit how outbound calls and API version policies are handled in a project.

For some teams, this becomes a natural moment to revisit the role of RestTemplate, RestClient, WebClient, Feign-like patterns, and broader API design standards.

### Checkpoints

- decide whether current outbound HTTP patterns are still consistent
- review whether older client styles should be phased down
- define an explicit API versioning strategy instead of relying on conventions alone

## 5. Observability is now harder to ignore

Spring Boot 4.0 continues moving toward more operations-friendly defaults. One of the clearest signs is the introduction of an OpenTelemetry starter, along with continued improvements in tracing and metrics-related configuration.

This matters because teams increasingly need to answer operational questions quickly:

- where is latency coming from?
- which external dependency is slowing things down?
- where did a failure start?
- how should traces and metrics be viewed together?

### Checkpoints

- review whether the team is Micrometer-first, OpenTelemetry-first, or hybrid
- review renamed tracing-related properties
- validate log, metric, and trace integration in a real environment

## 6. Testing dependencies need more explicit structure

Testing is another area where Spring Boot 4.0 becomes more explicit. Technology-specific test starters matter more, and test infrastructure is less likely to behave correctly by accident.

That is especially relevant in projects with different kinds of tests:

- security tests
- MVC slice tests
- integration tests
- external API tests

### Checkpoints

- verify whether technology-specific test dependencies are correctly separated
- re-check security test setup if `spring-security-test` is involved
- make sure integration and slice tests are clearly distinguished

## 7. The real risk is not new features — it is old assumptions

The hardest part of a major release is often not the shiny new capability. It is the quiet disappearance of old habits.

Spring Boot 4.0 includes several changes that affect more than application code:

- Undertow support removal
- removed or reworked integrations
- removal of embedded launch scripts for fully executable jars
- operational and packaging assumptions that may no longer hold

That means the migration may touch deployment scripts, CI/CD pipelines, operational guides, and infrastructure conventions as much as the code itself.

## A practical migration approach

If I were planning a real production upgrade, I would approach it like this:

1. stabilize on the latest reliable 3.x state first
2. review dependencies and starters before touching application code
3. audit configuration files carefully
4. strengthen tests before the actual upgrade
5. validate observability and operational behavior in a realistic environment

The most important point is that a green build is not enough. Regression confidence matters much more.

## Final thought

The biggest story of Spring Boot 4.0 is not “more features.” It is that the framework keeps moving toward a more explicit, more modular, and more operations-aware structure.

That makes it a valuable release — but also one that deserves real preparation. For teams already running Spring Boot 3.x in production, the smarter move is not to rush the version bump. It is to use the migration as an opportunity to revisit dependencies, configuration, test design, and operations together.

Done well, Spring Boot 4.0 can be more than an upgrade. It can be a useful reset toward a cleaner structure.
