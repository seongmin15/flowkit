# Idempotency & Rate Limit

## 개요

FlowKit은 Webhook 기반 트리거를 통해 외부 시스템 이벤트를 수신한다.
중복 요청 방지를 위한 멱등성 처리와, 트래픽 보호를 위한 Rate Limit을 핵심 보호 전략으로 적용한다.

---

## 멱등성 (Idempotency)

### 목적

동일한 이벤트가 여러 번 전달되더라도 Workflow Run이 중복 생성되지 않도록 보장한다.

### 구현 방식

- **Redis 기반 중복 검증**
- 중복 판별 키: `eventId` 또는 `payload hash`
- 수신 시 Redis에 키 존재 여부를 확인하여 중복 요청 차단
- TTL 설정을 통해 일정 시간 후 키 자동 만료

### 처리 흐름

```
Webhook 요청 수신
    │
    ▼
eventId / payload hash 추출
    │
    ▼
Redis 조회 → 키 존재? ─── Yes → 409 Conflict (중복 차단)
                │
                No
                │
                ▼
        Redis에 키 저장 (TTL 설정)
                │
                ▼
        Workflow Run 생성
```

---

## Rate Limit

### 목적

Webhook 엔드포인트에 대한 과도한 요청을 제한하여 시스템을 보호한다.

### 적용 단위

- **워크스페이스(Workspace) 단위**: 조직/팀별 요청 제한
- **엔드포인트(Endpoint) 단위**: 개별 Webhook URL별 요청 제한

### 초과 시 처리

| 전략 | 설명 |
|------|------|
| 429 응답 | 즉시 거부하고 `429 Too Many Requests` 반환 |
| 큐잉 처리 | 초과 요청을 큐에 적재하여 순차 처리 |

### 구현 방식

- Redis 기반 Sliding Window 또는 Token Bucket 알고리즘 적용
- 요청 수/시간 윈도우를 설정 기반으로 관리
