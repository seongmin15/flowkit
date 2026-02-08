# flowkit — Low-code Automation Platform (DAG-based)

본 프로젝트는 **비개발자도 복잡한 자동화 흐름을 설계하고 실제 운영 환경에서 안정적으로 실행할 수 있는
Low-code 자동화 플랫폼**을 구현하는 것을 목표로 한다.

본 프로젝트는 단순한 기능 구현이 아니라,
**직무 분석을 통해 드러난 부족 요소(운영 경험, 장애 대응, 확장성, 관측성, CI/CD)를
프로젝트 설계와 구현을 통해 체계적으로 보완하는 것**을 1차 목표로 한다.

플랫폼 내부 실행 모델은 DAG(Directed Acyclic Graph) 기반이며,
사용자는 DAG 구조를 이해하지 않고도
단계(Step), 조건(Condition), 승인(Approval) 중심의 흐름으로 자동화를 정의할 수 있다.

---

## 프로젝트 목표

- 비개발자용 Low-code 자동화 플랫폼 구현
- DAG 기반 실행 엔진 설계 및 구현
- Webhook 기반 이벤트 수신 및 보호 전략 검증
- 비동기 실행, 재시도, 장애 복구 메커니즘 구현
- 이벤트/로그/메트릭 기반 운영 가시성 확보
- Kubernetes + CI/CD + GitOps 운영 흐름 검증
- 직무 분석에서 요구되는 실서비스 운영 역량 강화

---

## 프로젝트 구조

```
flowkit/
├── docs/                    # 아키텍처, 운영, 설계 문서
│   ├── architecture/        # 시스템 전반 구조 및 핵심 설계
│   ├── adr/                 # Architecture Decision Records
│   ├── runbooks/            # 장애 대응 및 운영 시나리오
│   └── roadmap/             # 확장 로드맵
├── services/                # 런타임 애플리케이션 서비스
│   ├── workflow-api/        # Workflow/Run 관리, Webhook 트리거
│   ├── workflow-engine/     # DAG 해석, Task 상태 전이, 재시도
│   ├── worker/              # Stateless Task 실행 워커
│   └── integration-hub/     # 이벤트 기반 외부 시스템 연동
├── packages/                # 공통 라이브러리
│   ├── flowkit-core/        # 도메인 모델 및 상태 머신
│   ├── flowkit-contracts/   # 이벤트 스키마 및 DTO
│   ├── flowkit-observability/ # 메트릭, 로그, 트레이싱
│   ├── flowkit-security/    # 인증/인가 공통 모듈
│   └── flowkit-testkit/     # 테스트 유틸리티
├── infra/                   # 인프라 및 배포 구성
│   ├── docker/
│   ├── k8s/                 # base / dev / prod
│   ├── helm/
│   └── gitops/
├── tools/                   # 개발 및 운영 보조 도구
│   ├── db-migration/
│   ├── load-test/
│   └── scripts/
├── api-spec/                # API 명세
│   ├── openapi/
│   └── asyncapi/
└── .github/
```

---

## 1. Workflow 관리

- Workflow 생성 / 수정 / 버전 관리
- DAG(Node / Edge) 기반 정의
- Workflow 실행(Run) 생성
- Run 생성 시 특정 Workflow 버전 고정 (Immutable Run)

---

## 2. Trigger

- Webhook 기반 트리거
- 외부 시스템 이벤트 수신
- Redis 기반 멱등성 처리
  - eventId / payload hash 기반 중복 방지
- Rate Limit을 통한 트래픽 보호
  - 워크스페이스/엔드포인트 단위 제한
  - 초과 시 429 또는 큐잉 처리

---

## 3. Execution Engine

- 비동기 Task 실행
- Task 상태 관리 (PENDING → RUNNING → SUCCESS | FAILED | RETRYING)
- Task 재시도(backoff) 정책
- Worker Stateless 설계
- Worker 수평 확장 고려 설계

---

## 4. Event & Logging

- Workflow / Run / Task lifecycle 이벤트 발행
- Kafka 기반 이벤트 스트림
- Logstash → Elasticsearch → Kibana 로그 파이프라인
- runId 기준 이벤트 타임라인 추적
- At-least-once 이벤트 전달 및 소비자 멱등성 고려

---

## 5. Observability

- Prometheus 기반 메트릭 수집
- Grafana 대시보드 구성
  - API RPS
  - API Latency (p95)
  - Task 처리량 및 적체량
  - 실패율 및 재시도 횟수

---

## 6. Integration Hub

- 플랫폼 이벤트 구독
- Notion / GPT / Slack 등 외부 시스템 연동
- 설정 기반 라우팅
  - eventType → destination[]
  - 코드 수정 없이 연동 확장 가능 구조

---

## 아키텍처 개요

### 전체 구성

- **workflow-api** — Workflow / Run 관리, Webhook 수신 및 보호 로직
- **workflow-engine** — 비동기 Task 실행 엔진
- **worker** — Step 실행 워커
- **Kafka** — 이벤트 버스
- **ELK Stack** — 로그 및 이벤트 분석
- **Prometheus / Grafana** — 운영 지표 모니터링
- **Integration Hub** — 외부 시스템 연동

### 아키텍처 스타일

- Hexagonal Architecture (Ports & Adapters)
- Event-driven Architecture
- Infrastructure 의존성 분리

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

## 테스트 전략

- 단위 테스트
  - Domain / Application Layer
- 통합 테스트
  - PostgreSQL
  - Redis
  - Kafka
- Testcontainers 기반 실제 인프라 환경 검증
- Webhook 중복 처리 / 재시도 / 장애 시나리오 테스트

---

## CI/CD & GitOps

- Jenkins 기반 CI
  - 빌드 및 단위 테스트
  - Testcontainers 기반 통합 테스트
  - Docker 이미지 빌드 및 레지스트리 푸시
- GitOps Repository 업데이트
- Argo CD 기반 CD
  - Kubernetes 리소스 동기화
  - dev 자동 배포 / prod 승인 기반 배포
- 롤백 및 배포 이력 관리

---

## 운영 / 장애 대응 시나리오

- Webhook 트래픽 폭주로 인한 스파이크
- Worker 장애로 인한 Task 적체
- Kafka 지연 / 메시지 유실
- Redis 장애 시 Graceful Degradation
- 장애 발생 후 로그 기반 원인 분석 및 재발 방지

---

## Non-Goals

- 자유도 높은 개발자용 스크립트 자동화
- 완전한 자연어 기반 자동화 (초기 범위 제외)
- UI 중심의 복잡한 시각적 DAG 편집기

---

## 향후 확장 방향

- 자연어 명령 → 자동화 Flow 생성
- 실행 데이터 기반 자동화 추천
- 조직 단위 템플릿 및 표준화

---

## 프로젝트 목적

본 프로젝트는 Low-code 자동화 플랫폼 구현을 통해,
**실제 서비스 운영 환경에서 요구되는 설계, 확장성, 관측성,
장애 대응, 배포 자동화 역량을 실전 수준으로 검증**하는 것을 목표로 한다.
