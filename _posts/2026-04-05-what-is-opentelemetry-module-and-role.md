---
title: "OpenTelemetry는 무엇이고 어떤 역할을 할까? Spring Boot 개발자가 알아야 할 구조 정리"
date: 2026-04-05 01:11:00 +0900
categories: [Observability]
tags: [opentelemetry, observability, tracing, metrics, otlp, spring boot]
description: "OpenTelemetry가 어떤 모듈이고 어떤 역할을 하는지, Spring Boot 개발자 관점에서 구조적으로 정리합니다."
toc: true
---

OpenTelemetry를 처음 보면 이름부터 약간 헷갈립니다.

- 라이브러리인가?
- 표준인가?
- 에이전트인가?
- tracing 도구인가?
- metrics 수집기인가?

사실 OpenTelemetry는 이 중 하나만으로 설명하기 어렵습니다. **관측 데이터(metrics, traces, logs)를 표준화된 방식으로 만들고, 전달하고, 연결하기 위한 생태계 전체**에 가깝기 때문입니다.

Spring Boot를 쓰는 입장에서는 특히 더 헷갈릴 수 있습니다. 이미 Micrometer, Actuator, Observation 같은 개념이 있는데, OpenTelemetry가 들어오면 역할이 겹쳐 보이기 때문입니다.

그래서 이 글에서는 비교보다 먼저, **OpenTelemetry 자체가 무엇이고 어떤 역할을 하는지**를 구조적으로 정리해보겠습니다.

> Spring Boot 4.0에서 Micrometer와 OpenTelemetry의 관계를 보고 싶다면, 이 글과 이어지는 **[Spring Boot 4.0에서 OpenTelemetry는 어떻게 동작할까? Micrometer와 차이까지 쉽게 정리](/posts/opentelemetry-vs-micrometer-in-spring-boot-4/)** 글을 함께 보면 흐름이 더 잘 잡힙니다.

## 먼저 결론부터

OpenTelemetry를 아주 짧게 정리하면 아래처럼 볼 수 있습니다.

- **OpenTelemetry는 관측 데이터의 표준 생태계**입니다.
- 애플리케이션 안에서 metrics, traces, logs를 다룰 수 있는 API/SDK를 제공합니다.
- 수집한 데이터를 OTLP 같은 표준 방식으로 외부 백엔드에 보낼 수 있게 합니다.
- 특정 벤더 종속 없이 observability 파이프라인을 설계할 수 있게 돕습니다.

즉, OpenTelemetry는 “대시보드 도구”가 아니라, **관측 데이터를 만들고 옮기는 공통 언어**에 가깝습니다.

## OpenTelemetry를 왜 다들 이야기할까

예전에는 관측 도구가 서로 많이 갈라져 있었습니다.

- tracing은 Zipkin/Jaeger
- metrics는 Prometheus
- logs는 ELK
- APM은 각 벤더 전용 에이전트

이렇게 도구가 나뉘다 보니, 애플리케이션 입장에서는 계측 방식도 제각각이 되기 쉬웠습니다.

OpenTelemetry가 중요한 이유는 이 문제를 줄여주기 때문입니다.

즉,

- 애플리케이션은 공통 방식으로 관측 데이터를 만들고
- 전송도 표준 방식으로 하고
- 백엔드는 필요에 따라 선택할 수 있게 하는 것

이게 OpenTelemetry가 주는 가장 큰 가치입니다.

## OpenTelemetry의 핵심 역할 4가지

### 1. 관측 데이터의 공통 모델 제공

OpenTelemetry는 metrics, traces, logs를 다루는 공통 모델을 제공합니다.

예를 들면 이런 것들입니다.

- trace / span
- metric instrument
- resource attributes
- context propagation

이 공통 모델이 중요한 이유는, 서비스가 많아질수록 “데이터를 어떤 이름과 형태로 볼 것인가”가 점점 중요해지기 때문입니다.

### 2. 애플리케이션 안에서 계측할 수 있는 API 제공

개발자는 OpenTelemetry API를 통해 직접 계측을 넣을 수 있습니다.

예를 들어:

- 특정 비즈니스 구간에 span 생성
- 특정 수치를 metric으로 기록
- 요청 흐름에 context 전파

다만 Spring Boot 실무에서는 무조건 OTel API를 직접 쓰기보다, 프레임워크가 이미 제공하는 관측 계층을 먼저 활용하는 경우가 많습니다.

### 3. SDK를 통해 실제 수집과 export 수행

API만으로는 데이터가 실제로 보내지지 않습니다. 실제 수집과 처리, export는 SDK가 맡습니다.

쉽게 나누면:

- **API**: 개발 코드가 붙는 표면
- **SDK**: 실제 처리 엔진

이 구분을 이해하면 OpenTelemetry 문서를 볼 때 훨씬 덜 헷갈립니다.

### 4. OTLP를 통해 외부 백엔드와 연결

OpenTelemetry가 널리 쓰이게 된 핵심 이유 중 하나가 바로 **OTLP(OpenTelemetry Protocol)** 입니다.

이 프로토콜 덕분에 애플리케이션은 특정 벤더 방식에 묶이지 않고, 비교적 표준화된 방식으로 관측 데이터를 외부로 전달할 수 있습니다.

즉,

- 앱에서 관측 데이터 생성
- Collector 또는 backend로 OTLP 전송
- Grafana, Tempo, Jaeger, vendor APM 등에서 활용

이런 그림이 가능합니다.

## OpenTelemetry를 구성하는 주요 요소

Spring Boot 개발자 관점에서 보면, 아래 요소들을 알아두면 대부분의 문서를 읽기 쉬워집니다.

### Resource

이 데이터가 **어떤 서비스에서 나온 것인지 설명하는 메타 정보**입니다.

예:
- service.name
- deployment.environment
- service.instance.id

나중에 observability backend에서 데이터를 찾을 때 굉장히 중요합니다.

### Tracer / Span

트레이싱의 핵심입니다.

- **Tracer**: span을 만드는 도구
- **Span**: 요청 흐름 안의 한 구간

예를 들어 HTTP 요청 하나 안에서:
- controller 진입
- DB 조회
- 외부 API 호출

이런 각각의 구간이 span이 될 수 있습니다.

### Meter / Metric

메트릭 관련 요소입니다.

- Counter
- Gauge
- Histogram 계열

다만 Spring Boot에서는 이 부분을 꼭 OpenTelemetry SDK로 직접 다루는 것보다, Micrometer를 중심으로 보는 경우가 많습니다.

### Context Propagation

분산 시스템에서 아주 중요합니다.

서비스 A에서 시작한 요청이 서비스 B, C로 흘러갈 때 같은 trace 문맥을 유지해야 “한 요청의 흐름”으로 볼 수 있습니다. 이걸 가능하게 하는 핵심이 context propagation입니다.

### Exporter

수집한 관측 데이터를 외부 시스템으로 보내는 역할입니다.

대표적으로:
- OTLP exporter
- Zipkin exporter 등

## OpenTelemetry는 도구가 아니라 ‘연결 구조’에 가깝다

많은 분들이 OpenTelemetry를 처음 접할 때, Jaeger나 Prometheus 같은 개별 제품처럼 생각합니다. 하지만 OpenTelemetry는 그런 종류의 단일 제품이 아닙니다.

오히려 더 정확한 표현은 이쪽입니다.

- 관측 데이터를 **어떻게 만들지**
- 어떤 형식으로 **표준화할지**
- 어떤 방식으로 **전달할지**

를 정의하고 구현하는 연결 구조

그래서 OpenTelemetry는 “보는 곳”이라기보다, **보낼 수 있게 만드는 층**이라고 이해하면 좋습니다.

## Spring Boot 개발자가 특히 알아야 할 포인트

### 1. OpenTelemetry를 안다고 해서 곧바로 Spring Boot observability 전체를 이해한 건 아니다

Spring Boot는 이미 Micrometer, Observation, Actuator 같은 강한 기존 축이 있습니다. 그래서 Spring Boot에서 OTel을 이해할 때는 “OTel 자체”와 “Spring이 OTel을 엮는 방식”을 나눠서 봐야 합니다.

### 2. traces / metrics / logs가 다 같은 성숙도로 붙는 건 아니다

이론적으로는 모두 OpenTelemetry 안에 있지만, 실제 프레임워크 지원 수준과 실무 활용 방식은 각각 다를 수 있습니다.

### 3. Resource와 export 경로를 먼저 이해하는 게 중요하다

처음엔 span 이름보다도, 서비스 이름과 환경 속성이 어떻게 붙고 어디로 export되는지를 보는 편이 더 실무적입니다.

## OpenTelemetry를 실무에 도입할 때 얻는 장점

### 벤더 종속성 완화

애플리케이션 코드가 특정 관측 벤더에 덜 묶이게 됩니다.

### 멀티 언어/멀티 서비스 환경에서 일관성 확보

Java만이 아니라 Go, Node.js, Python 등과 함께 가는 환경에서 훨씬 유리합니다.

### Collector 기반 확장성

애플리케이션이 직접 모든 백엔드로 보내지 않고, Collector를 통해 중간에서 가공/라우팅할 수 있습니다.

### tracing 중심 문제 분석에 강함

특히 분산 시스템이 커질수록 trace 기반 문제 추적 가치가 커집니다.

## 헷갈리지 않기 위한 한 줄 정리

OpenTelemetry를 한 줄로 정리하면 이렇습니다.

**“관측 데이터를 만들고, 표준화하고, 전달하는 공통 인프라”**

이 정도로 이해하면 이후 Spring Boot 문서나 Micrometer 문서를 볼 때 훨씬 덜 혼란스럽습니다.

## 마무리

OpenTelemetry는 단순히 “새로운 tracing 라이브러리”가 아닙니다. observability를 벤더별 기능이 아니라, **표준화된 데이터 흐름**으로 보게 만드는 큰 흐름의 일부입니다.

Spring Boot 개발자 입장에서는 이걸 먼저 이해한 뒤,

- Spring은 어디까지 직접 담당하는지
- Micrometer는 어떤 자리를 차지하는지
- Spring Boot 4.0에서는 어떤 연결이 강화됐는지

를 이어서 보는 게 가장 이해가 빠릅니다.

다음 글에서는 이 구조를 바탕으로, Spring Boot 4.0에서 **Micrometer와 OpenTelemetry가 실제로 어떻게 연결되는지**를 더 실무적으로 풀어볼 수 있습니다.

---

참고:
- [Spring Boot 4.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Release-Notes)
- [Spring Boot Observability Documentation](https://docs.spring.io/spring-boot/4.0-SNAPSHOT/reference/actuator/observability.html)
- [OpenTelemetry 공식 사이트](https://opentelemetry.io/)

이 초안은 2026-04 기준 공개된 공식 문서와 Spring Boot 문서를 바탕으로 정리했습니다.
