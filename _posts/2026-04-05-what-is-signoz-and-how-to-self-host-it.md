---
title: "SigNoz란 무엇인가? OpenTelemetry 관점에서 보는 특징과 셀프 호스팅 방법"
date: 2026-04-05 09:31:00 +0900
categories: [Observability]
tags: [signoz, observability, opentelemetry, self-hosted, docker, kubernetes]
description: "OpenTelemetry 기반 관측 환경에서 SigNoz를 추천하는 이유와 Docker/Kubernetes 기준 셀프 호스팅 방법을 함께 정리했습니다."
image:
  path: /assets/img/posts/signoz-self-hosting-cover.jpg
toc: true
---

> English version: [What Is SigNoz? Key Characteristics and Self-Hosting Approach from an OpenTelemetry Perspective](/posts/what-is-signoz-and-how-to-self-host-it-en/)

OpenTelemetry를 공부하다 보면 결국 한 지점에서 질문이 생깁니다.

- 수집은 어떻게 할까?
- Collector는 어디에 둘까?
- traces, metrics, logs는 어디서 같이 볼까?
- Grafana, Jaeger, Prometheus, Loki를 조합해야 할까?
- 아니면 처음부터 한 번에 볼 수 있는 도구가 있을까?

이 지점에서 자주 언급되는 도구 중 하나가 **SigNoz**입니다.

물론 모든 팀에게 무조건 정답인 도구는 아닙니다. 하지만 **OpenTelemetry 중심으로 observability를 이해하고, 직접 운영 가능한 환경까지 함께 보고 싶을 때 검토할 만한 선택지**인 것은 분명합니다.

이 글에서는 SigNoz가 어떤 도구인지, 어떤 상황에서 고려해볼 수 있는지, 그리고 셀프 호스팅은 어떻게 접근하면 되는지를 함께 정리해보겠습니다.

## 먼저 결론부터

SigNoz를 이해할 때 핵심은 아래 4가지입니다.

- **OpenTelemetry 친화적인 관측 도구**입니다.
- **traces / metrics / logs를 하나의 흐름으로 보기 쉽게 구성된 편**입니다.
- **셀프 호스팅 기준 진입장벽이 상대적으로 낮은 편**입니다.
- **초기 observability 스택을 너무 많이 쪼개지 않고 시작하기 좋은 선택지**가 될 수 있습니다.

즉, OpenTelemetry 기반 관측 환경을 처음 구성하는 팀이라면 한 번쯤 검토해볼 만한 도구입니다.

## SigNoz를 검토해볼 만한 이유

### 1. OpenTelemetry 중심 흐름과 잘 맞는다

SigNoz는 OpenTelemetry 기반 데이터 흐름과 잘 맞습니다.

즉,

- 애플리케이션에서 telemetry를 만들고
- Collector를 통해 받아서
- traces / metrics / logs를 연결해서 볼 수 있는 구조

를 비교적 자연스럽게 가져갈 수 있습니다.

요즘 observability를 새로 설계할 때는 특정 벤더 에이전트보다 **OTel 기반 표준 흐름**을 먼저 생각하는 경우가 많습니다. 그런 점에서 SigNoz는 방향성이 분명합니다.

### 2. 분리된 도구들을 억지로 이어붙이는 부담이 줄어든다

처음 observability 환경을 만들 때 흔히 드는 조합은 이런 식입니다.

- traces는 Jaeger
- metrics는 Prometheus
- dashboard는 Grafana
- logs는 Loki/ELK

이 조합은 강력하지만, 초반에는 생각보다 운영 포인트가 많습니다.

- 저장소는 어떻게 나눌지
- 데이터 연결은 어떻게 할지
- 화면은 어디서 볼지
- 팀원이 어디서 무엇을 봐야 하는지

이런 것들이 처음엔 꽤 번거롭습니다.

SigNoz는 이 부담을 줄여줍니다. 물론 내부적으로 복잡한 요소가 있지만, 사용자 입장에서는 **통합된 관측 경험**을 제공하려는 방향이 분명합니다.

### 3. 셀프 호스팅 진입이 비교적 쉽다

공식 문서 기준으로 SigNoz는 Docker Standalone과 Kubernetes 설치 가이드를 제공합니다.

특히 처음 시작할 때는 Docker Compose 기반으로 빠르게 올려보고, 이후 Kubernetes/Helm 기반으로 확장하는 흐름이 꽤 자연스럽습니다.

이 점이 좋습니다.

- 로컬이나 단일 서버에서 바로 체험 가능
- 팀 단위 PoC가 빠름
- 이후 운영 환경으로 단계적으로 옮기기 좋음

### 4. “처음부터 너무 어려운 observability”를 피할 수 있다

관측 환경은 잘못 시작하면 오히려 팀이 안 보게 됩니다.

- 도구가 너무 많고
- 구조가 너무 복잡하고
- 어디서 어떤 데이터를 봐야 할지 애매하면

결국 observability를 붙여놓고도 잘 활용하지 못하게 됩니다.

SigNoz는 적어도 초기에 **한 곳에서 흐름을 이해하기 쉽게** 만들어주는 장점이 있습니다.

## 그렇다고 SigNoz가 무조건 정답일까?

그건 아닙니다. 추천 글이라도 이 부분은 분명히 말하는 게 맞습니다.

### SigNoz가 잘 맞는 경우

- OpenTelemetry 중심 관측 환경을 만들고 싶을 때
- 셀프 호스팅을 고려할 때
- traces / metrics / logs를 함께 보고 싶을 때
- Grafana + Jaeger + Prometheus + Loki를 처음부터 따로 조립하기 부담스러울 때

### SigNoz가 덜 맞을 수 있는 경우

- 이미 Prometheus/Grafana/Tempo/Loki 체계가 잘 굴러가고 있을 때
- 특정 벤더 APM을 전사 표준으로 쓰고 있을 때
- 아주 세밀한 커스텀 구성과 분산 운영 체계가 이미 성숙해 있을 때

즉, SigNoz는 “항상 최고”라기보다 **특정 상황에서 시작점과 균형이 좋은 선택**이라고 보는 편이 맞습니다.

## SigNoz 셀프 호스팅은 어떻게 시작하면 좋을까

공식 문서 기준으로 가장 쉬운 시작은 **Docker Standalone**입니다.

### 1. Docker Standalone 방식

SigNoz 공식 가이드는 Docker Compose 기반 설치를 제공합니다.

기본 흐름은 대략 아래와 같습니다.

1. SigNoz 저장소 클론
2. `deploy` 디렉터리로 이동
3. Docker Compose로 실행

대표 예시는 아래 흐름입니다.

```bash
git clone -b main https://github.com/SigNoz/signoz.git
cd signoz/deploy/
cd docker
docker compose up -d --remove-orphans
```

이 방식은 아래처럼 시작하기 좋습니다.

- 로컬 테스트
- 단일 서버 PoC
- 소규모 내부 검증 환경

### 설치 전 체크 포인트

공식 문서 기준으로 특히 봐야 할 것은 아래입니다.

- Linux 또는 macOS 환경
- Docker / Docker Compose 준비
- Docker에 최소 4GB 메모리 할당
- 포트 `8080`, `4317`, `4318` 열려 있는지 확인

실제로 이 정도만 준비돼도 빠르게 대시보드를 띄워볼 수 있습니다.

### 설치 후 확인 포인트

- 컨테이너가 정상 실행되는지
- UI가 `http://localhost:8080` 또는 해당 서버 IP로 열리는지
- OTLP endpoint(`4317`, `4318`)로 데이터 수신 가능한지

공식 문서 예시 기준으로 collector, query-service, clickhouse 같은 주요 구성 요소가 정상 기동해야 합니다.

## Kubernetes에서는 어떻게 보나

운영 환경이 Kubernetes라면 보통 Helm 기반 설치를 고려하게 됩니다.

SigNoz 공식 문서도 Kubernetes 설치 가이드를 별도로 제공합니다. AWS, GCP, AKS, Local, ArgoCD 같은 플랫폼별 가이드를 나눠두고 있어, 운영 환경으로 확장할 때 방향을 잡기 좋습니다.

### Kubernetes에서의 장점

- 운영 환경과 더 자연스럽게 맞물림
- 확장성과 분리 운영이 쉬움
- Collector / OTLP / backend 연결을 더 체계적으로 설계 가능

### 주의할 점

- 처음부터 Kubernetes로 시작하면 진입장벽이 높아질 수 있음
- 스토리지, 리소스, 네트워크, retention 정책까지 같이 봐야 함
- PoC 단계라면 먼저 Docker 기반으로 이해를 잡는 편이 더 빠를 수 있음

제 경험상, 대부분은 아래 순서가 더 안전합니다.

1. Docker Compose로 개념/동작 확인
2. OTLP 수집 흐름 확인
3. Kubernetes/Helm으로 운영형 구조 확장

## OpenTelemetry와 연결할 때 구조는 어떻게 생각하면 좋을까

SigNoz를 쓸 때도 핵심은 도구 자체보다 **데이터 흐름**입니다.

대략 이렇게 생각하면 이해가 쉽습니다.

- 앱(Spring Boot, Java Agent, 수동 계측 등)에서 telemetry 생성
- OpenTelemetry Collector가 수집/가공
- SigNoz가 이를 저장/시각화

즉, SigNoz를 observability backend로 보고, Collector를 중간 허브로 두는 그림이 가장 자연스럽습니다.

이 구조가 좋은 이유는 다음과 같습니다.

- 애플리케이션과 backend를 느슨하게 분리할 수 있고
- Collector에서 batch, retry, filtering 등을 넣을 수 있고
- 나중에 backend를 바꿔도 앱 수정 범위를 줄일 수 있습니다.

## 추천하는 도입 순서

제가 추천하는 가장 현실적인 순서는 이렇습니다.

### 1단계. 로컬 또는 단일 서버에서 SigNoz 띄우기

먼저 대시보드와 기본 흐름을 눈으로 확인합니다.

### 2단계. Spring Boot 앱 하나 연결하기

- Java Agent 또는 Micrometer + OTLP export
- Collector를 경유해 전송
- traces / metrics가 실제로 들어오는지 확인

### 3단계. Collector 구성 다듬기

- batch
- memory limiter
- attributes 추가
- 샘플링 정책

### 4단계. 운영형 구조로 확장

- Kubernetes
- Helm
- retention 정책
- 스토리지와 리소스 계획

## 이런 경우라면 SigNoz를 검토해볼 수 있다

- OpenTelemetry를 공부하면서 실제로 눈으로 확인하고 싶은 분
- Jaeger/Prometheus/Grafana/Loki 조합이 아직 부담스러운 팀
- 셀프 호스팅 observability를 비교적 빠르게 시작하고 싶은 분
- traces / metrics / logs를 한 흐름에서 보고 싶은 분

## 마무리

SigNoz는 observability 세계에서 모든 팀에 동일하게 맞는 절대적 정답은 아닙니다. 다만 **OpenTelemetry 기반으로 직접 운영 가능한 관측 환경을 비교적 빠르게 이해하고 구성해보는 데 균형이 좋은 도구**인 것은 분명합니다.

특히 아래 조건이라면 충분히 검토해볼 가치가 있습니다.

- OpenTelemetry 친화적 구조를 원하고
- 셀프 호스팅을 고려하고
- 여러 도구를 처음부터 따로 조립하는 부담을 줄이고 싶을 때

이럴 때 SigNoz는 이해하기도 쉽고, 시작하기도 괜찮고, 확장 경로도 비교적 선명한 편입니다.

다음 단계로는 이 글과 이어서,
**Spring Boot 애플리케이션에서 SigNoz로 telemetry를 보내는 실제 설정 예제**까지 정리하면 흐름이 더 좋아집니다.

---

참고:
- [SigNoz Overview](https://signoz.io/docs/introduction/overview/)
- [SigNoz Docker Standalone Installation](https://signoz.io/docs/install/docker/)
- [SigNoz Kubernetes Installation](https://signoz.io/docs/install/kubernetes/)

이 초안은 2026-04 기준 공개된 SigNoz 공식 문서를 바탕으로 정리했습니다.
