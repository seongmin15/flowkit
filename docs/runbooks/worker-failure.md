# Runbook: Worker 장애로 인한 Task 적체

## 증상

- Task 상태가 PENDING/RUNNING에서 전이되지 않음
- Task 적체량(backlog) 메트릭 증가
- Worker Pod가 CrashLoopBackOff 또는 비정상 종료
- Workflow Run 완료 시간 지연

---

## 영향

- Workflow Run이 완료되지 않고 대기 상태 지속
- 후속 Task 및 의존 Workflow 실행 지연
- 사용자에게 자동화 실행 실패로 표시될 수 있음

---

## 진단

### 1. Worker Pod 상태 확인

```bash
# Pod 상태 확인
kubectl get pods -l app=worker

# 비정상 Pod 로그 확인
kubectl logs <pod-name> --tail=100

# Pod 이벤트 확인
kubectl describe pod <pod-name>
```

### 2. 메트릭 확인 (Grafana)

- Task 처리량 대시보드에서 처리 중단 시점 확인
- Task 적체량(backlog) 추이 확인
- 실패율 및 재시도 횟수 확인

### 3. Kafka 컨슈머 상태 확인

- Worker 컨슈머 그룹의 lag 확인
- 파티션별 처리 상태 확인

---

## 대응 절차

### 즉시 대응

1. **Pod 상태 복구**: CrashLoopBackOff인 경우 원인 확인 후 재시작
   ```bash
   kubectl rollout restart deployment worker
   ```
2. **수평 확장**: 정상 Worker가 부족한 경우 인스턴스 추가
   ```bash
   kubectl scale deployment worker --replicas=<N>
   ```
3. **Task 타임아웃 확인**: RUNNING 상태에서 멈춘 Task가 타임아웃 처리되는지 확인

### 안정화

4. Worker 복구 후 적체된 Task가 순차 처리되는 것을 확인
5. Task 상태 전이 정상화 모니터링 (PENDING → RUNNING → SUCCESS)
6. 재시도 정책에 의해 RETRYING 상태 Task 처리 확인

### 사후 조치

7. Worker 장애 원인 분석 (OOM, 외부 의존성 장애, 코드 버그 등)
8. 리소스 요청/제한(request/limit) 재검토
9. 필요 시 HPA(Horizontal Pod Autoscaler) 설정 조정
10. 장애 시나리오 테스트 추가
