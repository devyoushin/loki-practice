# Loki 로그 쿼리 실습

## 디렉토리 구조

```
loki-practice/
├── docs/
│   └── log-query-test.md
└── ops/
    └── config/
        ├── app/
        │   ├── deployment.yaml      # 다양한 형식의 로그를 생성하는 샘플 앱
        │   └── service.yaml         # 샘플 앱 Service
        └── loki/
            ├── loki-values.yaml     # Loki Helm values
            ├── promtail-values.yaml # Promtail Helm values
            └── alerting-rule.yaml   # 알림 규칙 예시
```

---

## 사전 조건

```bash
# Loki 스택 설치 확인
kubectl get pods -n monitoring

# 샘플 앱 배포
kubectl apply -f ../ops/config/app/deployment.yaml
kubectl apply -f ../ops/config/app/service.yaml

# Pod가 Running 상태인지 확인
kubectl get pods -l app=my-app
```

---

## Step 1: 기본 로그 조회

### Grafana Explore에서 확인

```bash
# Grafana 포트 포워딩
kubectl port-forward svc/grafana -n monitoring 3000:80
# 브라우저에서 http://localhost:3000 → Explore
```

### 기본 스트림 조회

```logql
# my-app의 모든 로그
{namespace="default", app="my-app"}

# 예상: info/warn/error 로그가 섞여서 출력
```

---

## Step 2: 라인 필터로 에러만 조회

```logql
# ERROR 키워드가 포함된 로그만
{namespace="default", app="my-app"} |= "ERROR"

# debug 로그 제외
{namespace="default", app="my-app"} != "DEBUG"

# 타임아웃 또는 연결 거부 에러
{namespace="default", app="my-app"} |~ "timeout|connection refused"
```

예상 결과:
```
2024-01-10T12:00:01Z ERROR DB connection failed: timeout
2024-01-10T12:00:05Z ERROR Request failed: connection refused
```

---

## Step 3: JSON 파싱 후 필드 기반 필터

샘플 앱은 JSON 형식으로도 로그를 출력합니다.

```logql
# JSON 파싱 후 level이 error인 것만
{namespace="default", app="my-app"} | json | level = "error"

# 응답 시간 1초 초과
{namespace="default", app="my-app"} | json | duration_ms > 1000

# HTTP 5xx 에러
{namespace="default", app="my-app"} | json | status_code >= 500
```

---

## Step 4: 메트릭 쿼리 (시계열 데이터)

```logql
# 초당 에러 발생률
rate({namespace="default", app="my-app"} |= "ERROR" [5m])

# 앱별 초당 로그 발생률 비교
sum by (app) (rate({namespace="default"}[1m]))
```

Grafana에서 Time series 패널로 추가하면 그래프로 시각화됩니다.

---

## Step 5: P99 응답 시간 계산

```logql
quantile_over_time(0.99,
  {namespace="default", app="my-app"}
  | json
  | unwrap duration_ms [5m]
)
```

예상 결과: 99번째 백분위수 응답 시간 (숫자 시계열)

---

## Step 6: 알림 규칙 적용

```bash
kubectl apply -f ../ops/config/loki/alerting-rule.yaml

# 알림 규칙 확인
kubectl port-forward svc/loki -n monitoring 3100:3100
curl http://localhost:3100/loki/api/v1/rules | python3 -m json.tool
```

에러를 강제로 유발해서 알림이 발생하는지 확인합니다.

```bash
# 에러 로그를 반복 생성 (앱에 따라 다름)
kubectl exec -it deployment/my-app -- sh -c \
  "for i in \$(seq 1 20); do echo '{\"level\":\"error\",\"message\":\"DB connection failed\",\"duration_ms\":3000}'; sleep 0.5; done"
```

---

## Step 7: 실시간 로그 모니터링 (Live Tail)

```bash
# Grafana Explore → "Live" 버튼 클릭
# 또는 LogCLI 사용
```

```bash
# logcli 설치 (선택)
brew install logcli  # macOS

# 실시간 로그 tail
logcli query --addr=http://localhost:3100 \
  '{namespace="default", app="my-app"}' --tail
```

---

## 쿼리 성능 팁

| 상황 | 권장 쿼리 패턴 |
|------|---------------|
| 특정 앱 로그 확인 | `{namespace="...", app="..."}` — 스트림 범위를 좁게 |
| 전체 에러 파악 | `{namespace="..."} \|= "error"` — 라인 필터 먼저 |
| JSON 파싱 후 필터 | 라인 필터(`\|= "error"`)로 줄인 뒤 `\| json` 적용 |
| 시계열 그래프 | `rate()` + `sum by (레이블)` 조합 |

---

## 자주 발생하는 문제

| 증상 | 원인 | 해결 |
|------|------|------|
| "No data" | 시간 범위가 잘못됨 | Grafana 우상단 시간 범위 조정 |
| 쿼리 속도 느림 | 스트림 선택자가 너무 광범위 | 레이블로 범위 좁히기 |
| JSON 파싱 실패 | 로그가 JSON 형식이 아님 | 실제 로그 형식 확인 후 logfmt 또는 regexp 파서 시도 |
| 레이블 없음 | Promtail relabel_configs 누락 | Promtail 설정 및 타겟 상태 확인 |
