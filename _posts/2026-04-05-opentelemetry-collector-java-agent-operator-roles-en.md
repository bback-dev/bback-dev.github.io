---
title: "What Do OpenTelemetry Collector, Java Agent, and Operator Actually Do? A Simple Structural Guide"
date: 2026-04-05 01:22:00 +0900
categories: [Observability]
tags: [opentelemetry, collector, java agent, operator, kubernetes, observability]
description: "A practical guide to the roles of OpenTelemetry Collector, Java Agent, and Operator, including DaemonSet, sidecar, and gateway patterns."
image:
  path: /assets/img/posts/otel-collector-agent-operator-cover.jpg
toc: true
---

> Korean version: [OpenTelemetry Collector, Java Agent, Operator는 각각 무슨 역할을 할까? 환경별 구조 쉽게 정리](/posts/opentelemetry-collector-agent-operator-roles/)

Once you study OpenTelemetry for even a short time, the terminology starts piling up quickly.

- Collector
- Java Agent
- Operator
- DaemonSet
- Sidecar
- Gateway
- Exporter
- Processor
- Receiver

At first, all of these can feel like variations of the same thing. But if you do not separate their roles clearly, observability architecture stays blurry — especially once Kubernetes enters the picture.

So this post focuses on one simple goal: explain what **Collector**, **Java Agent**, and **Operator** actually do, and why deployment patterns such as **DaemonSet**, **sidecar**, and **gateway** keep appearing.

## Quick summary

A very simple breakdown looks like this:

- **Java Agent**: extracts telemetry automatically from inside the application
- **Collector**: receives, processes, and exports telemetry as a pipeline hub
- **Operator**: manages OpenTelemetry-related deployment and configuration in Kubernetes

Or even shorter:

- the **Agent creates data**
- the **Collector gathers and forwards data**
- the **Operator manages the setup in Kubernetes**

That mental model already clears up most confusion.

## Why separate the roles at all?

Telemetry work involves more steps than people first expect.

- data has to be created inside the application
- context has to propagate across services
- telemetry must be sent safely over the network
- data may need filtering, batching, or enrichment
- deployment patterns vary by environment

If an application tried to do all of this directly on its own, the result would become messy very quickly. That is why the responsibilities are split.

## 1. Java Agent: automatic instrumentation without code changes

The Java Agent is one of the most common entry points into OpenTelemetry for Java teams.

Its job is straightforward:

**attach an agent JAR at runtime and automatically instrument supported framework and library operations**.

In practice, that means you can often collect telemetry for things such as:

- incoming HTTP requests
- outbound HTTP calls
- database access
- messaging operations

without editing application code directly.

### When Java Agent is a good fit

- when you want tracing quickly
- when you want minimal code changes
- when you want common instrumentation across many services

### Limitations of Java Agent

- business-specific custom spans are still limited compared with manual instrumentation
- some details may be harder to fine-tune
- automatic instrumentation is convenient, but it can be harder to interpret if the team does not understand what is happening underneath

So the Java Agent is especially strong for **fast adoption** and **shared baseline instrumentation**.

## 2. Collector: much more than a relay server

A lot of people first think of the Collector as just a middle box. In practice, it is far more important than that.

The Collector is a **telemetry pipeline hub**.

Its role is to:

- receive telemetry from many services
- process it if needed
- export it to one or more backends

That makes application design much more flexible.

### Why Collector matters in practice

- applications can send telemetry quickly and stay simpler
- retries, batching, filtering, and enrichment can be centralized
- changing vendors becomes easier later
- telemetry can be fanned out to multiple destinations

Once service count grows, this becomes a significant operational advantage.

## The three core Collector parts: Receiver, Processor, Exporter

If you want to understand Collector well, these three concepts matter most.

### Receiver

A **Receiver** is the input.

Examples:

- OTLP receiver
- Prometheus receiver
- Jaeger receiver

This is how telemetry enters the Collector.

### Processor

A **Processor** modifies or manages telemetry in the middle of the pipeline.

Examples:

- batching
- memory limiting
- attribute enrichment
- sensitive-data filtering
- sampling

Processors are where Collector becomes operationally powerful rather than just passive.

### Exporter

An **Exporter** is the output path.

Examples:

- OTLP exporter
- Jaeger exporter
- Prometheus exporter
- vendor-specific exporters

A simple analogy is:

- Receiver = entrance
- Processor = workbench
- Exporter = exit

## Where do you deploy a Collector?

The Collector’s role stays the same, but deployment patterns change depending on operational goals.

### 1. Sidecar pattern

A Collector runs next to the application in the same Pod.

#### Advantages
- very close to the app
- short network path
- easier per-service isolation

#### Trade-offs
- the number of Collectors grows with the number of Pods
- operational overhead can rise quickly
- config management can become repetitive

This is useful when per-service control matters more than fleet simplicity.

### 2. DaemonSet pattern

A Collector runs once per Kubernetes node.

This is common for node-level collection and agent-like telemetry routing.

#### Advantages
- one Collector per node is simpler to reason about
- good for host metrics, logs, and node-local collection
- applications can send to a nearby local Collector

#### Trade-offs
- less fine-grained per-service separation
- scaling follows node count rather than service boundaries

A common real-world model is:

**node-local DaemonSet Collector -> central gateway Collector -> backend**

### 3. Gateway pattern

A separate central Collector cluster acts as a shared telemetry hub.

For example:

- apps or node-local collectors send to a gateway collector
- gateway collectors enforce shared policies and forward to backends

#### Advantages
- centralized control is easier
- routing, sampling, and vendor branching are easier to manage
- backend changes become simpler

#### Trade-offs
- it can become a central bottleneck if not designed well
- availability and scaling need to be planned more carefully

In real environments, **agent-like collectors plus gateway collectors** are often combined.

## 3. Operator: the Kubernetes manager for all of this

The Operator is not part of the telemetry pipeline itself.

Instead, it is a **Kubernetes management layer** for OpenTelemetry resources and deployment patterns.

In simple terms, it helps manage things like:

- Collector deployment creation and updates
- auto-instrumentation setup
- OpenTelemetry configuration through Kubernetes-native resources

So the Operator is not a replacement for Collector. It is an automation layer for running and managing OpenTelemetry in Kubernetes.

### Why Operator is useful

Without it, teams may end up manually managing:

- Collector deployments
- config maps
- sidecar or injection patterns
- auto-instrumentation annotations
- version updates

That becomes tedious quickly in clusters with many services or teams.

### When Operator is especially valuable

- in clusters with many applications
- when auto-instrumentation should be managed consistently
- when OpenTelemetry setup should be handled declaratively

## Java Agent vs Operator

These two are often confused, but their roles are very different.

- **Java Agent** performs runtime instrumentation
- **Operator** manages how that instrumentation and the Collector setup are deployed in Kubernetes

So the Operator does not replace the Agent. In many cases, it helps automate the Agent’s injection and lifecycle.

## Common environment patterns

### Local development or small experiments

- app + Java Agent
- app sends directly to a backend or a local Collector

This is the fastest and simplest way to start.

### VM or traditional server environments

- app + Java Agent
- local or separate Collector

In this case, Operator usually does not matter.

### Small to medium Kubernetes setups

- Java Agent or manual instrumentation
- DaemonSet Collector or simple gateway Collector
- Operator if management starts getting repetitive

### Large Kubernetes environments

- Operator manages Collector and instrumentation setup
- DaemonSet Collectors handle local collection
- gateway Collectors handle central routing and policy
- multiple backends or vendor paths may be supported

This tends to scale better operationally.

## The simplest way to remember it

If you want one clean memory aid, use this:

### Java Agent
“automatic instrumentation inside the app”

### Collector
“the telemetry pipeline hub for receive, process, and export”

### Operator
“the Kubernetes automation and management layer”

Once you keep those roles separate, the architecture gets much easier to understand.

## Common misunderstandings

### “If I have a Collector, I do not need an Agent”
Not necessarily. The Agent creates telemetry inside the app; the Collector handles it afterward.

### “If I use Operator, I do not need Collector”
No. Operator manages resources; Collector is still the actual telemetry pipeline.

### “DaemonSet is always the correct answer”
No. It is one strong pattern, but sidecar, gateway, and hybrid approaches can all be right depending on the environment.

## Final thought

OpenTelemetry architecture feels complex not because there are too many tools, but because the responsibilities are intentionally separated.

That is actually a good thing.

Once you map the roles clearly:

- Java Agent = instrumentation
- Collector = receive / process / export
- Operator = Kubernetes management

and inside Collector:

- Receiver
- Processor
- Exporter

then the whole structure becomes much easier to reason about.

That foundation makes later topics — such as how a Spring Boot app connects to Java Agent, Micrometer, Collector, and an OTLP backend — much easier to follow.
