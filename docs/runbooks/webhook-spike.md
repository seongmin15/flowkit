# Runbook: Webhook 트래픽 폭주 (Spike)

## 증상

- Webhook 엔드포인트로 비정상적으로 많은 요청 유입
- API 응답 시간 증가 (Latency p95 급등)
- 429 Too Many Requests 응답 비율 급증
- workflow-api Pod CPU/Memory 사용량 급증

---

## 영향

- 정상 Webhook 요청이 지연되거나 거부됨
- Workflow Run 생성 지연
- 하류 시스템(workflow-engine, worker)에 부하 전파 가능

---

## 진단

### 1. 메트릭 확인 (Grafana)

- API RPS 대시보드에서 요청량 급증 확인
- API Latency (p95) 이상 여부 확인
- Rate Limit 429 응답 비율 확인

### 2. 로그 확인 (Kibana)

- workflow-api 로그에서 요청 소스(IP, workspace) 확인
- Rate Limit 적용 로그 확인
- 에러 로그 패턴 분석

### 3. 인프라 확인

```bash
# Pod 상태 확인
kubectl get pods -l app=workflow-api

# Pod 리소스 사용량
kubectl top pods -l app=workflow-api

# HPA 상태 확인
kubectl get hpa workflow-api
```

---

## 대응 절차

### 즉시 대응

1. **Rate Limit 강화**: 해당 워크스페이스/엔드포인트의 Rate Limit 임계값 하향 조정
2. **수평 확장**: workflow-api Pod 수 증가
   ```bash
   kubectl scale deployment workflow-api --replicas=<N>
   ```
3. **트래픽 차단**: 악의적 소스로 판단 시 IP/워크스페이스 단위 차단

### 안정화

4. Kafka를 통한 비동기 처리로 백프레셔 완화 확인
5. 하류 시스템(workflow-engine, worker) 정상 동작 확인
6. 메트릭이 정상 범위로 돌아오는지 모니터링

### 사후 조치

7. 트래픽 폭주 원인 분석 (외부 시스템 오류, 재시도 폭풍 등)
8. Rate Limit 임계값 재검토 및 조정
9. 필요 시 큐잉 처리 전략 도입 검토
