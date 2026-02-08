# ADR-0001: Workflow Versioning

## 상태

승인됨 (Accepted)

## 날짜

2026-02-08

---

## 컨텍스트

FlowKit에서 Workflow는 사용자가 정의한 자동화 흐름이다.
운영 환경에서 Workflow는 지속적으로 수정될 수 있으며,
실행 중인 Run이 Workflow 변경에 영향을 받으면 예측 불가능한 동작이 발생할 수 있다.

다음 문제를 해결해야 한다:
- 실행 중인 Run이 Workflow 수정에 영향받지 않아야 함
- 특정 시점의 Workflow 상태를 재현할 수 있어야 함
- 변경 이력을 추적할 수 있어야 함

---

## 결정

**Workflow Version을 별도 엔티티로 관리하고, Run 생성 시 특정 Version을 고정(Immutable Run)한다.**

### 설계 원칙

1. **Workflow**: 자동화 흐름의 논리적 식별자 (이름, 메타데이터)
2. **Workflow Version**: Workflow의 특정 시점 스냅샷 (DAG 정의 포함, Immutable)
3. **Run**: 특정 Workflow Version에 바인딩된 실행 인스턴스

### 동작 방식

- Workflow 수정 시 새로운 Version이 생성됨
- Run 생성 시 지정된 (또는 최신) Version에 고정됨
- Run 실행 중 Workflow가 수정되어도 해당 Run에는 영향 없음

---

## 결과

### 장점

- 실행 중인 Run의 안정성 보장
- 실행 재현성 확보 (동일 Version으로 동일 결과)
- 감사(Audit) 추적 및 롤백 용이
- 버전 간 비교(diff) 가능

### 단점

- Version 엔티티 추가로 인한 데이터 모델 복잡도 증가
- 저장 공간 증가 (Version별 DAG 스냅샷 보관)

### 대안 (기각됨)

- **Workflow 직접 참조**: Run이 Workflow를 직접 참조하면 수정 시 실행 중인 Run이 영향받음
- **Soft Copy**: Run 생성 시 DAG를 복사하면 Version 관리 없이도 격리 가능하지만, 변경 이력 추적 불가
