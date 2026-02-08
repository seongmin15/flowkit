# Architecture Overview

## 개요

FlowKit은 DAG(Directed Acyclic Graph) 기반의 Low-code 자동화 플랫폼이다.
비개발자가 단계(Step), 조건(Condition), 승인(Approval) 중심의 흐름으로 자동화를 정의하고,
실제 운영 환경에서 안정적으로 실행할 수 있도록 설계되었다.

---

## 전체 구성

| 컴포넌트 | 역할 |
|----------|------|
| **workflow-api** | Workflow / Run 관리, Webhook 수신 및 보호 로직 |
| **workflow-engine** | 비동기 Task 실행 엔진 |
| **worker** | Step 실행 워커 |
| **Kafka** | 이벤트 버스 |
| **ELK Stack** | 로그 및 이벤트 분석 |
| **Prometheus / Grafana** | 운영 지표 모니터링 |
| **Integration Hub** | 외부 시스템 연동 |

---

## 아키텍처 스타일

- **Hexagonal Architecture (Ports & Adapters)**: 도메인 로직을 외부 인프라로부터 분리
- **Event-driven Architecture**: Kafka 기반 이벤트 스트림을 통한 비동기 통신
- **Infrastructure 의존성 분리**: 인프라 구현체를 어댑터로 분리하여 교체 가능하도록 설계

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| Backend | Spring Boot, Java 17 |
| Database | PostgreSQL |
| Cache | Redis |
| Messaging | Kafka |
| Observability | Prometheus, Grafana |
| Logging | Logstash, Elasticsearch, Kibana |
| Test | JUnit5, Testcontainers |
| Container | Docker |
| Orchestration | Kubernetes |
| CI/CD | Jenkins, GitOps, Argo CD |

---

## 서비스 간 흐름

```
[Webhook 요청]
    │
    ▼
workflow-api (수신 → 멱등성 검증 → Rate Limit)
    │
    ▼
workflow-engine (DAG 해석 → Task 생성 → 상태 전이)
    │
    ▼
worker (Task 실행 → 결과 보고)
    │
    ▼
Kafka (이벤트 발행)
    │
    ├──▶ ELK Stack (로그 수집 및 분석)
    ├──▶ Prometheus / Grafana (메트릭 모니터링)
    └──▶ Integration Hub (외부 시스템 연동)
```
