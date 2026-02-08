# Execution Model

## 개요

FlowKit의 실행 모델은 DAG(Directed Acyclic Graph) 기반이다.
Workflow는 Node(Step)와 Edge(의존 관계)로 구성되며,
workflow-engine이 DAG를 해석하여 Task 실행 순서를 결정하고,
worker가 각 Task를 비동기로 실행한다.

---

## Workflow 구조

- **Workflow**: 자동화 흐름의 최상위 단위
- **Workflow Version**: Workflow의 특정 시점 스냅샷 (Immutable)
- **Run**: Workflow Version을 기반으로 생성된 실행 인스턴스
- **Task**: Run 내에서 실행되는 개별 작업 단위

---

## DAG 기반 정의

- Workflow는 Node(Step)와 Edge(의존 관계)로 구성
- 각 Node는 실행 가능한 Step, 조건 분기(Condition), 승인 대기(Approval) 등의 타입을 가짐
- Edge는 Node 간 선후 관계를 정의하며, DAG 구조를 통해 병렬 실행 가능 여부를 판단

---

## Task 상태 머신

```
PENDING → RUNNING → SUCCESS
                  → FAILED
                  → RETRYING → RUNNING
```

| 상태 | 설명 |
|------|------|
| PENDING | Task 생성 후 실행 대기 |
| RUNNING | Worker에 의해 실행 중 |
| SUCCESS | 정상 완료 |
| FAILED | 최종 실패 (재시도 소진) |
| RETRYING | 재시도 대기 중 |

---

## Task 재시도 정책

- Exponential Backoff 기반 재시도
- 최대 재시도 횟수 설정 가능
- 재시도 소진 시 FAILED 상태로 전이

---

## Worker 설계 원칙

- **Stateless**: Worker는 상태를 보유하지 않으며, Task 실행에 필요한 모든 정보를 메시지로 수신
- **수평 확장**: Worker 인스턴스 수를 늘려 처리량 증가 가능
- **독립 실행**: 각 Worker는 독립적으로 Task를 수신하고 실행

---

## Immutable Run

- Run 생성 시 특정 Workflow Version을 고정
- Run 실행 중 Workflow가 수정되어도 해당 Run에는 영향 없음
- 실행 재현성 및 감사 추적 보장
