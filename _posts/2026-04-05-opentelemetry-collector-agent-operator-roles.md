---
title: "OpenTelemetry Collector, Java Agent, Operator는 각각 무슨 역할을 할까? 환경별 구조 쉽게 정리"
date: 2026-04-05 01:22:00 +0900
categories: [Observability]
tags: [opentelemetry, collector, java agent, operator, kubernetes, observability]
description: "OpenTelemetry Collector, Java Agent, Operator가 각각 어떤 역할을 하는지, DaemonSet 같은 배포 방식까지 실무 관점에서 쉽게 정리합니다."
image:
  path: /assets/img/posts/otel-collector-agent-operator-cover.jpg
toc: true
---

> English version: [What Do OpenTelemetry Collector, Java Agent, and Operator Actually Do? A Simple Structural Guide](/posts/opentelemetry-collector-java-agent-operator-roles-en/)

OpenTelemetry를 조금 공부하다 보면 곧바로 용어가 폭발합니다.

- Collector
- Java Agent
- Operator
- DaemonSet
- Sidecar
- Gateway
- Exporter
- Processor
- Receiver

처음에는 전부 비슷해 보이는데, 사실 이걸 구분하지 못하면 observability 아키텍처가 머릿속에 잘 안 잡힙니다. 특히 Kubernetes 환경까지 들어가면 더 헷갈립니다.

그래서 이 글에서는 **Collector / Java Agent / Operator가 각각 무엇을 하는지**, 그리고 **DaemonSet, sidecar, gateway 같은 배포 방식이 왜 나오는지**를 한 번에 정리해보겠습니다.

> OpenTelemetry 자체가 어떤 모듈이고 어떤 역할을 하는지 먼저 보고 싶다면, **[OpenTelemetry는 무엇이고 어떤 역할을 할까? Spring Boot 개발자가 알아야 할 구조 정리](/posts/what-is-opentelemetry-module-and-role/)** 글을 먼저 보는 것도 좋습니다.

## 먼저 결론부터

OpenTelemetry 구성을 아주 단순하게 나누면 이렇게 보면 됩니다.

- **Java Agent**: 애플리케이션 안에서 관측 데이터를 자동으로 뽑아내는 도구
- **Collector**: 관측 데이터를 받아서 가공하고 외부 백엔드로 보내는 중간 허브
- **Operator**: Kubernetes에서 Collector와 자동 계측 설정을 관리해주는 운영 도구

즉,

- **Agent는 데이터를 만든다**
- **Collector는 데이터를 모으고 다듬고 보낸다**
- **Operator는 Kubernetes에서 이 구성을 관리한다**

이렇게 이해하면 대부분의 구조가 정리됩니다.

## 왜 역할을 나눠야 할까

관측 데이터는 생각보다 일이 많습니다.

- 앱 안에서 수집해야 하고
- 서비스 간 문맥도 이어야 하고
- 네트워크로 안전하게 보내야 하고
- 백엔드에 맞게 변환하거나 필터링해야 하고
- 환경별로 배포도 달라져야 합니다.

이 모든 걸 애플리케이션 하나가 직접 다 하게 만들면 너무 복잡해집니다. 그래서 역할을 나눕니다.

## 1. Java Agent: 코드 수정 없이 계측을 붙이는 도구

Java Agent는 OpenTelemetry를 접할 때 가장 많이 보게 되는 진입점 중 하나입니다.

역할은 단순합니다.

**애플리케이션 실행 시 에이전트 JAR를 붙여서, 프레임워크와 라이브러리 호출 지점에 자동 계측을 주입하는 것**입니다.

쉽게 말하면:

- Spring Boot 앱 코드 직접 수정 없이
- HTTP 요청
- 외부 HTTP 호출
- DB 호출
- 메시징

같은 지점에서 telemetry를 자동 수집하게 도와줍니다.

### Java Agent가 좋은 경우

- 빠르게 tracing을 붙여보고 싶을 때
- 코드 변경을 최소화하고 싶을 때
- 운영 환경에서 공통 계측 기준을 맞추고 싶을 때

### Java Agent의 한계

- 비즈니스 의미가 담긴 커스텀 span은 한계가 있음
- 애플리케이션 구조에 따라 수집 결과를 더 세밀하게 조정하기 어려울 수 있음
- "자동"이라서 편하지만, 왜 이런 span이 생겼는지 내부를 이해하지 못하면 해석이 어려울 수 있음

즉, Java Agent는 **빠른 시작과 공통 계측**에 강합니다.

## 2. Collector: 데이터를 받는 것에서 끝나지 않는 중간 허브

Collector는 많은 분들이 처음엔 그냥 “중계 서버” 정도로 생각합니다. 하지만 실제로는 훨씬 더 중요합니다.

Collector는 기본적으로 **telemetry pipeline의 허브** 역할을 합니다.

즉,

- 여러 서비스에서 telemetry를 받고
- 필요한 가공을 하고
- 하나 이상의 backend로 내보냅니다.

그래서 앱이 직접 backend로 쏘는 구조보다 훨씬 유연해집니다.

### Collector가 주는 장점

- 앱은 데이터만 빨리 넘기면 됨
- retry, batch, filtering, enrichment 등을 중앙에서 처리 가능
- 벤더 변경 시 앱 수정 범위를 줄일 수 있음
- 여러 backend로 fan-out 가능

실무에서는 이 차이가 꽤 큽니다.

예를 들어 서비스 수가 많아지면, 각 서비스가 vendor API에 직접 붙는 구조보다 Collector를 사이에 두는 구조가 훨씬 관리하기 편해집니다.

## Collector 안의 핵심 구성요소: Receiver / Processor / Exporter

Collector를 이해할 때 가장 중요한 건 이 3개입니다.

### Receiver

데이터를 **받는 입구**입니다.

예:
- OTLP receiver
- Prometheus receiver
- Jaeger receiver

즉, telemetry가 Collector 안으로 들어오는 지점입니다.

### Processor

받은 데이터를 **중간에서 다듬는 단계**입니다.

예:
- batch 처리
- memory limiter
- attributes 추가/수정
- 민감정보 제거
- sampling

Processor는 실무에서 매우 중요합니다. 단순 전달이 아니라, 운영에 맞는 형태로 telemetry를 정리해주는 지점이기 때문입니다.

### Exporter

최종적으로 데이터를 **밖으로 내보내는 출구**입니다.

예:
- OTLP exporter
- Jaeger exporter
- Prometheus exporter
- vendor-specific exporter

즉,

**Receiver는 입구, Processor는 가공대, Exporter는 출구**라고 보면 이해가 쉽습니다.

## Collector는 어디에 배포하나

여기서부터 Kubernetes/운영 환경 이야기가 나옵니다.

Collector는 역할은 같아도, 배포 방식에 따라 운영 성격이 달라집니다.

### 1. Sidecar 패턴

애플리케이션 Pod 옆에 Collector를 같이 붙이는 방식입니다.

#### 장점
- 앱과 가장 가깝다
- 네트워크 홉이 짧다
- 서비스별로 독립 제어 가능

#### 단점
- Pod 수가 많아질수록 Collector도 같이 늘어남
- 운영 비용이 커질 수 있음
- 설정 관리가 번거로워질 수 있음

즉, 세밀한 제어에는 좋지만 대규모에선 무거울 수 있습니다.

### 2. DaemonSet 패턴

Kubernetes의 각 노드마다 Collector를 하나씩 띄우는 방식입니다.

이 패턴은 특히 node-level 수집이나 agent 성격 배포에서 자주 나옵니다.

#### 장점
- 노드당 하나라 배포 구조가 단순함
- 로그/호스트 메트릭/노드 단위 데이터 수집에 적합
- 애플리케이션이 가까운 로컬 Collector로 보낼 수 있음

#### 단점
- 서비스별 세밀한 분리가 어렵다
- 노드 수에 맞춰 확장되므로 중앙 집중형 제어와는 성격이 다름

실무적으로는 **DaemonSet Collector가 노드 근처에서 1차 수집**을 맡고, 이후 gateway Collector로 보내는 구조도 자주 씁니다.

### 3. Gateway 패턴

Collector를 중앙 수집 허브처럼 따로 두는 방식입니다.

예를 들어:
- app / sidecar / daemonset → gateway collector → backend

#### 장점
- 중앙에서 정책 통제하기 좋음
- 샘플링, 라우팅, 벤더 분기 관리에 유리
- backend 변경 시 유연함

#### 단점
- 중앙 병목이 될 수 있음
- 고가용성/확장 설계가 필요함

실무에서는 **agent-like collector + gateway collector** 조합이 자주 나옵니다.

## 3. Operator: Kubernetes에서 이 모든 걸 관리해주는 관리자

Operator는 Collector처럼 데이터를 직접 처리하는 컴포넌트가 아닙니다.

역할은 **Kubernetes에서 OpenTelemetry 관련 리소스를 선언형으로 관리하는 것**입니다.

쉽게 말하면:

- Collector 배포 생성/관리
- 자동 계측(auto-instrumentation) 주입 관리
- Kubernetes 리소스와 OTel 설정 연결

즉, Operator는 “데이터 파이프라인 실행기”가 아니라 **운영 자동화 도구**입니다.

### Operator가 필요한 이유

Kubernetes 환경에서는 수작업으로:

- Collector Deployment 만들고
- ConfigMap 만들고
- sidecar 붙이고
- auto-instrumentation annotation 붙이고
- 버전 올리고

이걸 다 직접 하다 보면 금방 복잡해집니다.

Operator는 이 복잡도를 줄여줍니다.

### Operator가 특히 빛나는 경우

- 클러스터에 여러 팀/여러 서비스가 있을 때
- Java / Node / Python auto-instrumentation 주입을 통제하고 싶을 때
- Collector 배포 방식을 선언형으로 관리하고 싶을 때

## Java Agent와 Operator는 어떻게 다를까

이 둘도 자주 헷갈립니다.

- **Java Agent**는 실제 계측을 붙이는 런타임 도구
- **Operator**는 Kubernetes에서 그 에이전트 주입과 Collector 배포를 관리하는 관리자

즉, Operator가 Java Agent를 "대체"하는 게 아닙니다.

오히려 Kubernetes에서는 Operator가 **Java Agent 자동 주입을 관리하는 역할**을 할 수 있습니다.

## 환경별로 어떤 조합이 많이 쓰일까

### 로컬 개발 / 소규모 테스트

- 앱 + Java Agent
- 앱 → 직접 backend 또는 로컬 Collector

가장 단순하고 빠릅니다.

### VM / 전통 서버 환경

- 앱 + Java Agent
- 로컬 또는 별도 Collector

이 경우 Operator는 보통 필요 없습니다.

### Kubernetes 중소규모 환경

- Java Agent 또는 수동 계측
- DaemonSet Collector 또는 간단한 Gateway Collector
- 필요 시 Operator 도입

### Kubernetes 대규모 환경

- Operator로 Collector/auto-instrumentation 관리
- 노드 단위 DaemonSet 수집
- 중앙 Gateway Collector로 라우팅
- backend 다중 전송 / 정책 통제

이 구조가 가장 확장성이 좋습니다.

## 실무적으로 어떻게 이해하면 좋을까

제 기준에서는 이렇게 보면 제일 덜 헷갈립니다.

### Java Agent

“앱 안에서 telemetry를 뽑아내는 빠른 자동 계측 도구”

### Collector

“telemetry를 받아서 정리하고 전달하는 파이프라인 허브”

### Operator

“Kubernetes에서 Collector와 자동 계측 구성을 선언형으로 관리하는 운영 도구”

이 3개를 섞어 생각하지 않으면 구조가 훨씬 선명해집니다.

## 자주 나오는 오해

### “Collector가 있으면 Agent는 필요 없나?”

아닙니다. Agent는 데이터를 **만들고**, Collector는 그 데이터를 **받고 처리**합니다. 둘은 대체 관계가 아니라 보완 관계입니다.

### “Operator가 있으면 Collector를 대신하나?”

아닙니다. Operator는 관리 도구이고, Collector는 실제 데이터 파이프라인입니다.

### “무조건 DaemonSet이 정답인가?”

그렇지 않습니다. DaemonSet은 좋은 패턴 중 하나일 뿐이고, sidecar / gateway / 혼합형이 모두 상황에 따라 가능합니다.

## 마무리

OpenTelemetry 구성이 복잡하게 보이는 이유는 도구가 많아서가 아니라, **역할이 나뉘어 있기 때문**입니다.

하지만 반대로 생각하면, 역할만 분리해서 이해하면 구조는 오히려 단순해집니다.

- Java Agent는 계측
- Collector는 수집/가공/전달
- Operator는 Kubernetes 운영 자동화

그리고 Collector 내부에서는 다시

- Receiver
- Processor
- Exporter

이 3단으로 보면 됩니다.

이 구조를 이해하고 나면, 문서를 볼 때도 “이게 어디 역할이지?”가 훨씬 빨리 보이기 시작합니다.

다음 단계에서는 이 흐름을 바탕으로, Spring Boot 애플리케이션이 실제로 **Java Agent / Micrometer / Collector / OTLP backend**와 어떻게 이어지는지 예제로 보는 글까지 자연스럽게 연결할 수 있습니다.

---

참고:
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)
- [OpenTelemetry Operator for Kubernetes](https://opentelemetry.io/docs/platforms/kubernetes/operator/)
- [OpenTelemetry Java Agent](https://opentelemetry.io/docs/zero-code/java/agent/)

이 초안은 2026-04 기준 공개된 OpenTelemetry 공식 문서를 바탕으로 정리했습니다.
