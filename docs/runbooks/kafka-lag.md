# Runbook: Kafka 지연 / 메시지 유실

## 증상

- Kafka 컨슈머 그룹 lag 지속 증가
- 이벤트 처리 지연 (이벤트 발행 후 소비까지 시간 증가)
- ELK 로그 파이프라인에 이벤트 누락
- Integration Hub 외부 연동 지연

---

## 영향

- Workflow/Run/Task lifecycle 이벤트 처리 지연
- 로그 기반 모니터링 실시간성 저하
- 외부 시스템 연동(Slack, Notion 등) 알림 지연
- 이벤트 타임라인 추적 불완전

---

## 진단

### 1. Kafka 클러스터 상태 확인

```bash
# 브로커 상태 확인
kubectl get pods -l app=kafka

# 토픽 상태 확인
kafka-topics.sh --describe --topic <topic-name> --bootstrap-server <broker>

# 컨슈머 그룹 lag 확인
kafka-consumer-groups.sh --describe --group <group-id> --bootstrap-server <broker>
```

### 2. 메트릭 확인 (Grafana)

- Kafka lag 메트릭 대시보드 확인
- 프로듀서/컨슈머 처리량(throughput) 확인
- 브로커별 파티션 분포 확인

### 3. 컨슈머 애플리케이션 확인

```bash
# 컨슈머 Pod 로그 확인
kubectl logs -l app=workflow-engine --tail=100
kubectl logs -l app=integration-hub --tail=100
```

---

## 대응 절차

### 즉시 대응

1. **컨슈머 상태 확인**: 컨슈머 Pod가 정상 동작하는지 확인
2. **컨슈머 재시작**: 컨슈머가 멈춘 경우 재시작
   ```bash
   kubectl rollout restart deployment <consumer-app>
   ```
3. **컨슈머 확장**: lag이 처리량 부족인 경우 인스턴스 추가
   ```bash
   kubectl scale deployment <consumer-app> --replicas=<N>
   ```

### 메시지 유실 대응

4. **At-least-once 확인**: 프로듀서 acks 설정 확인 (acks=all 권장)
5. **소비자 멱등성 확인**: 중복 이벤트가 안전하게 처리되는지 확인
6. **오프셋 관리**: 컨슈머 오프셋 커밋 전략 확인 (수동 커밋 권장)

### 안정화

7. lag이 점진적으로 감소하는지 모니터링
8. 이벤트 타임라인에서 누락 이벤트 확인
9. 필요 시 누락 이벤트 재발행

### 사후 조치

10. Kafka 브로커 리소스 및 파티션 수 재검토
11. 컨슈머 처리 성능 병목 분석
12. 파티션 리밸런싱 전략 검토
13. lag 기반 알림 임계값 조정
