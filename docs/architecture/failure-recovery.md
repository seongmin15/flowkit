# Failure Recovery

## 개요

FlowKit은 운영 환경에서 발생할 수 있는 다양한 장애 시나리오에 대한 복구 전략을 설계한다.
비동기 실행, 재시도, Graceful Degradation을 통해 시스템 안정성을 확보한다.

---

## 장애 시나리오 및 대응 전략

### 1. Webhook 트래픽 폭주 (Spike)

- **증상**: 외부 시스템에서 대량의 Webhook 요청이 동시에 유입
- **대응**:
  - Rate Limit을 통한 요청 제한 (429 응답 또는 큐잉)
  - 워크스페이스/엔드포인트 단위 트래픽 격리
  - Kafka를 통한 비동기 처리로 백프레셔 완화

### 2. Worker 장애로 인한 Task 적체

- **증상**: Worker 인스턴스 장애로 Task가 처리되지 않고 큐에 적체
- **대응**:
  - Worker Stateless 설계로 인스턴스 교체 가능
  - Kubernetes 기반 자동 재시작 및 수평 확장
  - Task 타임아웃 설정으로 점유 방지
  - 적체 메트릭 모니터링 및 알림

### 3. Kafka 지연 / 메시지 유실

- **증상**: Kafka 브로커 장애 또는 컨슈머 지연으로 이벤트 처리 지연
- **대응**:
  - At-least-once 이벤트 전달 보장
  - 소비자 측 멱등성 처리로 중복 이벤트 안전 처리
  - Kafka lag 메트릭 모니터링
  - 파티션 리밸런싱 및 컨슈머 그룹 관리

### 4. Redis 장애 시 Graceful Degradation

- **증상**: Redis 장애로 멱등성 검증 및 Rate Limit 동작 불가
- **대응**:
  - Redis 장애 감지 시 fallback 전략 적용
  - 멱등성 검증 일시 비활성화 또는 DB 기반 대체
  - Rate Limit 완화 또는 보수적 기본값 적용
  - Redis 복구 후 자동 전환

---

## Task 재시도 (Retry)

- Exponential Backoff 기반 재시도 정책
- 최대 재시도 횟수 초과 시 FAILED 상태 전이
- 재시도 횟수 메트릭 수집 및 모니터링

---

## 장애 분석 및 재발 방지

- `runId` 기준 이벤트 타임라인 추적
- ELK Stack 기반 로그 분석
- Grafana 대시보드를 통한 실패율/재시도 추이 확인
- 장애 발생 후 Runbook 기반 원인 분석 및 재발 방지 조치
