# Loki 알림 가이드

Loki는 로그를 기반으로 알림을 발생시킬 수 있습니다. Prometheus의 PrometheusRule 리소스와 동일한 방식으로 설정합니다.

---

## 동작 원리

```
[Loki]
  │  LogQL 메트릭 쿼리를 주기적으로 실행 (ruler)
  │
  ▼
[Alertmanager]
  │  알림 라우팅 (Slack, PagerDuty, Email 등)
  │
  ▼
[알림 수신]
```

> Loki의 `ruler` 컴포넌트가 알림 규칙을 평가합니다. Loki values에서 ruler를 활성화해야 합니다.

---

## 1. Loki Ruler 활성화

```yaml
# loki-values.yaml
loki:
  rulerConfig:
    storage:
      type: local
      local:
        directory: /etc/loki/rules
    rule_path: /tmp/loki/scratch
    alertmanager_url: http://alertmanager.monitoring:9093
    ring:
      kvstore:
        store: inmemory
    enable_api: true
```

---

## 2. 알림 규칙 작성

### PrometheusRule 리소스로 작성

```yaml
# loki/alerting-rule.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: loki-alerts
  namespace: monitoring
  labels:
    role: alert-rules
spec:
  groups:
    - name: loki-log-alerts
      rules:
        # 에러 로그 급증 알림
        - alert: HighErrorRate
          expr: |
            sum(rate({namespace="default"} |= "error" [5m])) by (app)
            > 0.1
          for: 2m
          labels:
            severity: warning
          annotations:
            summary: "High error log rate detected"
            description: "App {{ $labels.app }} has error rate {{ $value | humanize }}/s for more than 2 minutes."

        # 특정 에러 메시지 발생 알림
        - alert: DatabaseConnectionError
          expr: |
            count_over_time({namespace="default"} |= "DB connection failed" [5m]) > 5
          for: 1m
          labels:
            severity: critical
          annotations:
            summary: "Database connection errors detected"
            description: "More than 5 DB connection failures in the last 5 minutes."

        # 로그가 전혀 안 오는 경우 (앱 다운 감지)
        - alert: NoLogsReceived
          expr: |
            sum(rate({namespace="default", app="my-app"}[5m])) == 0
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "No logs received from my-app"
            description: "my-app has not produced any logs for 5 minutes."
```

```bash
kubectl apply -f loki/alerting-rule.yaml
```

---

## 3. 알림 규칙 작성 패턴

### 패턴 1: 에러율 임계값 초과

```yaml
- alert: ErrorRateHigh
  expr: |
    sum(rate({namespace="default"} |= "ERROR" [5m])) by (app)
    /
    sum(rate({namespace="default"}[5m])) by (app)
    > 0.05
  for: 3m
  labels:
    severity: warning
  annotations:
    summary: "Error rate exceeds 5%"
```

### 패턴 2: 특정 키워드 등장 횟수

```yaml
- alert: OutOfMemoryError
  expr: |
    count_over_time(
      {namespace="default"} |= "OutOfMemoryError" [10m]
    ) > 3
  for: 0m
  labels:
    severity: critical
```

### 패턴 3: JSON 파싱 후 숫자 기반 알림

```yaml
- alert: SlowResponse
  expr: |
    quantile_over_time(0.95,
      {namespace="default", app="my-app"}
      | json
      | unwrap duration_ms [5m]
    ) > 2000
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "P95 response time exceeds 2 seconds"
```

---

## 4. 알림 상태 확인

```bash
# Loki ruler API로 알림 규칙 확인
kubectl port-forward svc/loki -n monitoring 3100:3100
curl http://localhost:3100/loki/api/v1/rules

# 현재 발생 중인 알림 확인
curl http://localhost:3100/loki/api/v1/alerts

# Alertmanager UI 접속
kubectl port-forward svc/alertmanager -n monitoring 9093:9093
# 브라우저에서 http://localhost:9093 접속
```

---

## 5. Grafana에서 알림 설정 (대안)

Loki ruler 없이 Grafana의 Unified Alerting을 사용할 수도 있습니다.

```
Grafana → Alerting → Alert Rules → New alert rule
→ Query: LogQL 메트릭 쿼리 입력
→ Conditions: 임계값 설정
→ Contact point: Slack / Email 등 설정
```

---

## 참고

- [공식문서 - Loki Alerting](https://grafana.com/docs/loki/latest/alert/)
- [공식문서 - Ruler](https://grafana.com/docs/loki/latest/alert/ruler/)
- [공식문서 - Recording Rules](https://grafana.com/docs/loki/latest/alert/recording_rules/)
