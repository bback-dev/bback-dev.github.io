---
title: "Spring Boot 4.0에서 OpenTelemetry는 어떻게 동작할까? Micrometer와 차이까지 쉽게 정리"
date: 2026-04-05 01:01:00 +0900
categories: [Backend, Springboot]
tags: [opentelemetry, micrometer, spring boot 4, observability, tracing, metrics]
description: "Spring Boot 4.0에서 OpenTelemetry가 어떤 방식으로 동작하는지, Micrometer와 어떤 관계인지 실무 관점에서 쉽게 정리했습니다."
image:
  path: /assets/img/posts/micrometer-opentelemetry-spring-boot-cover.jpg
toc: true
---

> English version: [How OpenTelemetry Works in Spring Boot 4.0: A Practical View Alongside Micrometer](/posts/how-opentelemetry-works-with-micrometer-in-spring-boot-4-en/)

Spring Boot 4.0을 보다 보면 observability 쪽에서 눈에 띄는 변화가 하나 있습니다. 바로 **`spring-boot-starter-opentelemetry`**가 공식적으로 등장했다는 점입니다.

> OpenTelemetry 모듈 자체의 역할과 구성 요소를 먼저 보고 싶다면, 함께 읽을 글로 정리한 **[OpenTelemetry는 무엇이고 어떤 역할을 할까? Spring Boot 개발자가 알아야 할 구조 정리](/posts/what-is-opentelemetry-module-and-role/)** 를 먼저 보는 것도 좋습니다.

그런데 여기서 많은 분들이 바로 헷갈립니다.

- “이제 Spring Boot는 OpenTelemetry를 기본으로 쓰는 건가?”
- “그럼 Micrometer는 이제 안 쓰는 건가?”
- “metrics, tracing, logs가 전부 OpenTelemetry로 통일되는 건가?”

결론부터 말하면, **Spring Boot 4.0은 OpenTelemetry를 지원하지만, Spring 진영의 기본 관점은 여전히 Micrometer 중심**입니다. 다만 tracing과 export, 그리고 백엔드 연동 방식에서 OpenTelemetry와 연결되는 지점이 더 명확해졌습니다.

이 글에서는 그 관계를 실무적으로 헷갈리지 않게 정리해보겠습니다.

## 먼저 결론부터

Spring Boot 4.0에서 observability를 이해할 때 가장 중요한 포인트는 아래입니다.

- **메트릭의 기본 축은 여전히 Micrometer**입니다.
- **트레이싱은 Micrometer Observation/Tracing을 중심으로 OpenTelemetry와 연결될 수 있습니다.**
- **OpenTelemetry SDK 자체는 Spring Boot가 일부 bean과 resource를 구성해주지만, 모든 것을 자동으로 다 맡아주지는 않습니다.**
- **즉, “Spring Boot 4 = OpenTelemetry로 완전 전환”은 아닙니다.**

이 구조를 먼저 머리에 넣고 들어가면 대부분의 혼란이 줄어듭니다.

## Observability를 먼저 짧게 정리하면

observability는 대개 아래 3가지 축으로 이해하면 쉽습니다.

- **Metrics**: 수치 기반 상태 확인
  - 예: 요청 수, 응답 시간, 에러율, JVM 메모리
- **Tracing**: 요청 흐름 추적
  - 예: A 서비스 → B 서비스 → DB 호출까지 한 요청의 흐름
- **Logs**: 이벤트/문맥 기록
  - 예: 에러 로그, 비즈니스 로그, 디버그 로그

예전에는 이 셋을 서로 다른 도구로 따로 관리하는 경우가 많았습니다. 하지만 요즘은 한 요청의 흐름을 더 잘 연결해서 보기 위해, metrics / traces / logs를 더 유기적으로 다루는 방향으로 가고 있습니다.

그 흐름에서 OpenTelemetry가 점점 중요한 표준 역할을 하고 있고, Spring Boot 4.0도 이 흐름을 더 분명하게 받아들이고 있습니다.

## Micrometer와 OpenTelemetry는 경쟁 관계일까?

이 질문이 가장 많이 나옵니다.

짧게 말하면, **실무에서는 완전한 경쟁 관계라기보다 역할이 겹치면서도 관점이 다른 도구**라고 보는 편이 맞습니다.

### Micrometer는 무엇에 강한가

Micrometer는 Spring 생태계에서 **애플리케이션 메트릭 계측과 관측 추상화**에 매우 강합니다.

Spring Boot를 오래 써본 분들은 익숙할 겁니다.

- actuator metrics
- Timer, Counter, Gauge
- Prometheus export
- Observation API
- tracing bridge

즉, Spring Boot 입장에서는 Micrometer가 이미 아주 깊게 녹아 있는 상태입니다.

### OpenTelemetry는 무엇에 강한가

OpenTelemetry는 특정 프레임워크보다 더 넓은 차원에서, **관측 데이터의 표준화와 전송 방식**을 다루는 쪽에 가깝습니다.

쉽게 말하면 이런 느낌입니다.

- 여러 언어/플랫폼에서 공통 모델을 맞추고
- traces / metrics / logs를 표준 방식으로 다루고
- OTLP 같은 공통 프로토콜로 백엔드에 보내는 것

그래서 OpenTelemetry는 “표준”의 느낌이 강하고, Micrometer는 “Spring/Java 실무에서 잘 쓰이는 계측 도구”의 느낌이 강합니다.

## Spring Boot 4.0에서 기본 축은 왜 아직 Micrometer일까

공식 문서 기준으로 보면, Spring Boot 4.0은 OpenTelemetry를 지원하지만 **메트릭 수집의 기본 선택은 여전히 Micrometer**입니다.

이 말이 중요한 이유는, 많은 분들이 `starter-opentelemetry`가 생겼다는 이유만으로 “이제부터 metrics도 OTel SDK가 알아서 다 해주겠지”라고 생각하기 쉽기 때문입니다.

하지만 실제 구조는 조금 다릅니다.

- Spring Boot는 `OpenTelemetry` bean과 `Resource` bean 같은 기반 요소를 제공할 수 있고
- tracing은 Micrometer Tracing과 함께 OpenTelemetry export 방향으로 연결될 수 있지만
- **metrics 자체는 여전히 Micrometer registry를 통해 다루는 흐름이 기본**입니다.

즉, Spring Boot 4.0에서 메트릭을 보낸다고 할 때 실무적으로 더 자연스러운 경로는 보통 이쪽입니다.

**애플리케이션 → Micrometer → OTLP exporter → OpenTelemetry backend**

이 흐름이 핵심입니다.

## “그럼 OpenTelemetry starter는 정확히 뭘 해주나?”

Spring Boot 4.0 Release Notes 기준으로 새로 추가된 `spring-boot-starter-opentelemetry`는 OTLP export와 OpenTelemetry SDK 연결을 더 자연스럽게 만드는 starter입니다.

다만 여기서 중요한 점은, 이 starter를 넣었다고 해서 observability 전체가 자동 완성되는 것은 아니라는 점입니다. 이 starter의 더 구체적인 모듈 역할, Resource, SDK, export 지점은 별도 글에서 정리해두었습니다.

- [OpenTelemetry는 무엇이고 어떤 역할을 할까? Spring Boot 개발자가 알아야 할 구조 정리](/posts/what-is-opentelemetry-module-and-role/)

## Spring Boot 4.0에서 OpenTelemetry는 내부적으로 어떻게 이어질까

실무적으로 쉽게 이해하면 대략 아래 그림입니다.

### 1. 계측(Instrumentation)

애플리케이션 안에서 무엇을 측정할지 정합니다.

예:
- HTTP 요청 시간
- DB 호출 시간
- 외부 API 호출
- 배치 실행
- 메시지 처리

이 단계에서 Spring Boot는 여전히 Micrometer Observation 계열과 매우 밀접합니다.

### 2. 관측 데이터 생성

- metrics는 주로 Micrometer가 담당
- traces는 Observation/Tracing을 통해 span 형태로 이어짐

### 3. export

그 결과물을 OTLP 같은 방식으로 외부 백엔드(예: Tempo, Grafana, Jaeger, OpenTelemetry Collector, vendor backend)로 보냅니다.

즉, Spring Boot 4.0은 OpenTelemetry를 “직접 모든 것의 중심”으로 둔다기보다, **Spring 방식의 계측 결과를 OpenTelemetry 친화적인 경로로 내보내기 쉽게 만든다**고 보는 쪽이 더 정확합니다.

## 메트릭 관점에서 보면 왜 Micrometer가 여전히 중요할까

여기서 실무자가 가장 조심해야 할 부분이 있습니다.

공식 문서 기준으로 **Spring Boot는 OpenTelemetry의 SdkMeterProvider를 기본 메트릭 수집 수단으로 제공하지 않습니다.**

즉,

- OpenTelemetry API/SDK의 MeterProvider를 직접 쓰는 라이브러리가 있다면
- 그 메트릭은 Spring Boot 기본 구성만으로는 자동 export되지 않을 수 있습니다.

그래서 Spring 문서도 기본적으로는 이렇게 권장합니다.

- 메트릭은 **Micrometer로 수집/기록**하고
- 필요하면 **OTLP로 export**하라

이 접근이 좋은 이유는 다음과 같습니다.

- Spring Boot actuator와 자연스럽게 붙고
- 기존 메트릭 체계와 호환이 좋고
- 운영에서 보는 흐름이 안정적입니다.

즉, 메트릭 쪽에서는 “OTel을 쓴다”보다 **“Micrometer 메트릭을 OTel 호환 백엔드로 보낸다”**고 이해하는 편이 더 정확한 경우가 많습니다.

## 트레이싱 관점에서는 OpenTelemetry 체감이 더 크다

반대로 tracing은 OpenTelemetry를 더 직접적으로 체감하기 쉽습니다.

Spring Boot 4.0 문서에서도 tracing은 Micrometer Tracing과 함께 OpenTelemetry export 방향으로 연결되는 구조가 핵심입니다.

이걸 실무적으로 풀어 말하면 이렇습니다.

- 애플리케이션 요청이 들어오면 observation이 생성되고
- 이것이 trace/span으로 이어지고
- 그 trace가 OTLP나 Zipkin 등으로 전송됩니다.

과거에는 Brave/Zipkin 중심으로 설명되는 경우가 많았지만, 이제는 OpenTelemetry 쪽으로 표준이 더 모이는 분위기입니다.

그래서 Spring Boot 4.0에서 OpenTelemetry의 존재감은 **메트릭보다 tracing에서 더 직접적으로 느껴질 가능성**이 높습니다.

## 로그는 어떤가

로그는 많은 분들이 자연스럽게 “그럼 logs도 다 OpenTelemetry로 바로 되겠네?”라고 생각하는데, 여기서는 아직 조금 더 신중하게 보는 게 좋습니다.

Spring Boot 문서 기준으로도, OpenTelemetry 쪽 `SdkLoggerProvider` bean은 있을 수 있지만, **Spring Boot가 로그를 그쪽으로 자동 브릿지해주는 건 기본 제공하지 않습니다.**

즉, logs는 여전히 별도 log bridge나 로깅 구성, 수집 파이프라인이 필요할 수 있습니다.

그래서 실무에서 지금 시점 기준으로는 보통 이렇게 생각하는 게 안전합니다.

- metrics: Micrometer 중심
- traces: Micrometer Tracing + OTel export 체감 큼
- logs: 별도 전략 필요

## Spring Boot 4.0에서 OpenTelemetry를 쓸 때 헷갈리기 쉬운 포인트

### 1. “starter 추가 = 모든 observability 자동 완성”은 아니다

그렇지 않습니다. starter는 시작점이지, 운영 구성을 완성해주는 마법 버튼은 아닙니다.

### 2. MeterProvider를 직접 쓰는 라이브러리는 따로 봐야 한다

이 경우 기본 Spring Boot 메트릭 흐름 밖에 있을 수 있습니다.

### 3. Resource 설정은 중요하다

서비스 이름, 환경, 인스턴스 속성은 나중에 백엔드에서 데이터를 볼 때 굉장히 중요합니다.

Spring Boot는 `management.opentelemetry.resource-attributes`, `OTEL_RESOURCE_ATTRIBUTES`, `OTEL_SERVICE_NAME` 같은 방식으로 이 정보를 다룰 수 있습니다.

### 4. tracing export 관련 프로퍼티 이름 변화도 확인해야 한다

Spring Boot 4.0에서는 observability/tracing 관련 일부 프로퍼티 이름이 정리되었습니다. 마이그레이션 시에는 이 부분도 같이 봐야 합니다.

## 그럼 실무에서는 어떤 선택이 좋을까

제 기준에서는 아래처럼 가져가면 가장 덜 헷갈립니다.

### 경우 1. Spring Boot 기반 서비스이고 actuator를 적극 활용한다

이 경우는 **Micrometer 중심**으로 가는 게 가장 자연스럽습니다.

- 메트릭: Micrometer
- 트레이싱: Micrometer Observation/Tracing
- 백엔드 export: OTLP / Zipkin / vendor backend

### 경우 2. 전사적으로 OpenTelemetry 표준화를 강하게 밀고 있다

이 경우도 Spring Boot 애플리케이션 내부에서는 여전히 Micrometer를 완전히 버리기보다, **Spring 친화적인 계층은 유지하고 export와 통합 표준을 OTel로 맞추는 전략**이 현실적입니다.

### 경우 3. 특정 라이브러리가 OTel API를 직접 사용한다

이 경우는 MeterProvider/SDK wiring을 별도로 봐야 할 수 있습니다. 이 지점은 “Spring Boot 기본 자동 설정만으로 충분한가?”를 꼭 따져봐야 합니다.

## Spring Boot 4.0에서 observability를 설계할 때 추천하는 사고방식

저는 이걸 이렇게 정리하는 편이 가장 이해가 쉽다고 생각합니다.

- **Micrometer는 애플리케이션 관측의 실무 도구**
- **OpenTelemetry는 관측 데이터의 표준화와 연결 방식**
- **Spring Boot 4.0은 이 둘을 더 매끄럽게 잇는 방향으로 진화 중**

즉,

- Spring 안에서 계측은 여전히 Micrometer가 강하고
- 외부 observability 생태계와 연결될수록 OpenTelemetry 비중이 커진다고 보면 됩니다.

이 관점으로 보면 “Micrometer vs OpenTelemetry”보다 **“Micrometer와 OpenTelemetry를 어떻게 함께 이해할 것인가”**가 더 좋은 질문입니다.

## 마무리

Spring Boot 4.0에서 OpenTelemetry는 분명 더 중요한 위치로 올라왔습니다. 하지만 그렇다고 해서 Micrometer가 밀려난 것은 아닙니다.

오히려 실제 구조는 이쪽에 더 가깝습니다.

- Spring Boot 내부 관측: Micrometer 중심
- tracing/export 표준화: OpenTelemetry 친화적 방향 강화
- 운영 백엔드 연결: OTLP 중심 활용 확대

그래서 실무적으로 가장 좋은 접근은 “무조건 OTel로 갈아타야지”가 아니라,
**Spring Boot가 이미 잘하는 것(Micrometer)을 살리면서 OpenTelemetry와 자연스럽게 이어지는 구조를 이해하는 것**입니다.

이걸 이해하고 나면 Spring Boot 4.0의 observability 변화가 훨씬 덜 복잡하게 보입니다.

---

참고:
- [Spring Boot 4.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Release-Notes)
- [Spring Boot Observability Documentation](https://docs.spring.io/spring-boot/4.0-SNAPSHOT/reference/actuator/observability.html)
- [Spring Boot Tracing Documentation](https://docs.spring.io/spring-boot/4.0-SNAPSHOT/reference/actuator/tracing.html)

이 초안은 2026-04 기준 공개된 Spring Boot 4.0 문서와 릴리스 노트를 바탕으로 정리했습니다.
