---
title: "How OpenTelemetry Works in Spring Boot 4.0: A Practical View Alongside Micrometer"
date: 2026-04-05 01:01:00 +0900
categories: [Backend, Springboot]
tags: [opentelemetry, micrometer, spring boot 4, observability, tracing, metrics]
description: "A practical explanation of how OpenTelemetry fits into Spring Boot 4.0 and how it relates to Micrometer."
image:
  path: /assets/img/posts/micrometer-opentelemetry-spring-boot-cover.jpg
toc: true
---

> Korean version: [Spring Boot 4.0에서 OpenTelemetry는 어떻게 동작할까? Micrometer와 차이까지 쉽게 정리](/posts/opentelemetry-vs-micrometer-in-spring-boot-4/)

Spring Boot 4.0 makes observability conversations more interesting because OpenTelemetry is now much harder to ignore. The presence of `spring-boot-starter-opentelemetry` immediately raises a few common questions:

- does Spring Boot now use OpenTelemetry by default?
- does that mean Micrometer is no longer central?
- are metrics, traces, and logs all handled by OpenTelemetry now?

The short answer is no — at least not in the simplistic way many people first assume.

Spring Boot 4.0 supports OpenTelemetry, but the Spring ecosystem is still very much Micrometer-oriented in how it approaches application observability.

## Quick summary

The important points are:

- metrics still revolve around **Micrometer**
- tracing can be bridged through **Micrometer Observation / Tracing** into OpenTelemetry-friendly export paths
- Spring Boot can auto-configure some OpenTelemetry pieces, but it does not hand everything over to the OpenTelemetry SDK by default

So the right mental model is not “Spring Boot 4 equals full OpenTelemetry migration.” A better model is “Spring Boot 4 makes the Micrometer-to-OpenTelemetry path clearer.”

## Observability in three simple layers

It helps to split observability into three familiar areas:

- **metrics**: request counts, latency, error rates, JVM usage
- **traces**: end-to-end flow of a request across services and dependencies
- **logs**: contextual event records, failures, and runtime messages

Modern platforms increasingly try to connect these together. That is one reason OpenTelemetry matters so much today.

## Micrometer and OpenTelemetry are not a simple versus battle

In practice, they overlap, but they are not the same thing.

### What Micrometer is strong at

Micrometer is deeply integrated into the Spring world. It gives Spring Boot applications a familiar and practical way to instrument metrics and observations through concepts such as:

- actuator metrics
- timers, counters, and gauges
- Observation API
- tracing bridges
- multiple registry integrations

### What OpenTelemetry is strong at

OpenTelemetry feels broader. It focuses on standardized telemetry models, propagation, SDK behavior, and export protocols such as OTLP.

That is why OpenTelemetry often feels like a **cross-platform standard layer**, while Micrometer feels like a **very practical Spring-native observability layer**.

## Why Micrometer is still the default center for metrics

Official documentation still makes it clear that metrics collection in Spring Boot remains Micrometer-oriented.

That matters because many people see the new OpenTelemetry starter and assume that OpenTelemetry’s own meter provider now becomes the default path for application metrics. In Spring Boot 4.0, that is not the recommended default view.

A practical data path often still looks like this:

**application -> Micrometer -> OTLP exporter -> OpenTelemetry-compatible backend**

That is a more accurate description of how many Spring Boot 4.0 applications will work in reality.

## So what does the OpenTelemetry starter actually do?

The new `spring-boot-starter-opentelemetry` makes OTLP export and OpenTelemetry SDK integration more natural.

What it does **not** do is magically finish observability design for you.

It provides a stronger official path, but teams still need to think about:

- what gets instrumented
- where telemetry is exported
- how resources are identified
- how logs fit in
- what is handled by Micrometer vs what is delegated to OpenTelemetry components

If you want the broader conceptual side first, the companion post below is the better starting point:

- [What Is OpenTelemetry and What Role Does It Play?](/posts/what-is-opentelemetry-and-what-does-it-do-en/)

## A practical internal model for Spring Boot 4.0

A simple way to think about the flow is:

### 1. Instrumentation

The application decides what to observe:

- HTTP requests
- database access
- outbound HTTP calls
- scheduled tasks
- messaging operations

In Spring Boot, this stage is still closely tied to Micrometer Observation concepts.

### 2. Telemetry creation

- metrics are typically created through Micrometer
- traces are created through observation/tracing bridges into spans

### 3. Export

Telemetry is then exported through OTLP or other supported paths into systems such as Tempo, Jaeger, SigNoz, or vendor-specific backends.

That is why Spring Boot 4.0 should not be described as “OpenTelemetry all the way down.” It is more accurate to say that it makes Spring’s observability model more OpenTelemetry-friendly.

## Why metrics are still a special case

One subtle but important point is that Spring Boot does not treat OpenTelemetry’s `SdkMeterProvider` as the default metric pipeline for application metrics.

That means if a library uses OpenTelemetry’s meter API directly, those metrics may not automatically behave the same way as the Micrometer-driven metrics most Spring Boot users expect.

The practical recommendation remains:

- report application metrics through **Micrometer**
- export them through **OTLP** if needed

That keeps compatibility with Actuator, existing metric conventions, and typical operational expectations.

## Tracing is where OpenTelemetry feels more visible

Tracing is where many teams will feel OpenTelemetry more directly.

Spring Boot 4.0 keeps moving toward a world where traces flow more naturally through Micrometer Observation / Tracing into OpenTelemetry-compatible backends.

In practice that means:

- an observation starts around a request or operation
- that becomes one or more spans
- those spans are exported to a tracing backend

Historically, many Spring examples were framed around Brave and Zipkin. The trend today is far more OpenTelemetry-centered.

## What about logs?

Logs are still the area where people often overestimate how automatic everything is.

Spring Boot documentation makes it clear that although an OpenTelemetry logger provider can exist, Spring Boot does not automatically bridge regular application logs to it out of the box in the same way teams might expect.

So a safer current mental model is:

- metrics: Micrometer-first
- traces: Micrometer tracing with OpenTelemetry-friendly export
- logs: separate strategy still required

## Common misunderstandings

### “Adding the starter completes observability”
No. It improves the path, but design and operations still matter.

### “If a library uses OpenTelemetry API directly, everything is covered”
Not necessarily. Metrics and SDK wiring may still need extra care.

### “Resource metadata is secondary”
It is not. `service.name`, environment, and instance identity become extremely important once telemetry reaches a backend.

### “Property names never change much here”
Major versions are exactly when they do. Spring Boot 4.0 includes renamed tracing-related properties and related cleanup.

## A practical recommendation

For a typical Spring Boot service, the most realistic approach is still:

- instrument using Spring / Micrometer-friendly idioms
- use Micrometer for metrics
- use tracing bridges where appropriate
- export to OpenTelemetry-friendly backends through OTLP

That gives you the best of both worlds:

- Spring-native ergonomics inside the app
- OpenTelemetry-friendly interoperability outside the app

## Final thought

The most useful question is no longer “Micrometer or OpenTelemetry?” It is “how do they fit together inside a Spring Boot 4.0 application?”

Once you approach it that way, the structure becomes much clearer:

- Micrometer remains the most practical in-app observability layer
- OpenTelemetry becomes increasingly important as the standardization and transport layer
- Spring Boot 4.0 helps connect the two more smoothly than before

That is the shift worth understanding.
