# LogQL 심화 가이드

기초 필터링 이후 로그를 파싱하고, 메트릭으로 변환하는 방법을 다룹니다.

---

## 1. 로그 파싱 (Parsing)

파싱을 사용하면 로그 라인에서 구조화된 필드를 추출해 레이블처럼 사용할 수 있습니다.

### JSON 파싱

```logql
# JSON 형식 로그 → 모든 필드를 레이블로 추출
{app="my-app"} | json

# 특정 필드만 추출 (성능상 유리)
{app="my-app"} | json level, status_code, duration_ms

# 중첩 JSON 필드 추출
{app="my-app"} | json request_method="request.method", user_id="user.id"
```

예시 로그:
```json
{"level":"error","status_code":500,"duration_ms":1234,"message":"DB connection failed"}
```

파싱 후 필터링:
```logql
{app="my-app"} | json | level = "error" | duration_ms > 1000
```

### logfmt 파싱

```logql
# logfmt 형식: key=value key2=value2
{app="my-app"} | logfmt

# 예시 로그: level=info status=200 method=GET path=/api/users duration=42ms
{app="my-app"} | logfmt | status != "200"
```

### Pattern 파싱

로그 형식이 일정한 경우 패턴으로 추출합니다.

```logql
# Apache 접근 로그 예시: 127.0.0.1 - - [10/Jan/2024] "GET /api" 200 512
{app="nginx"} | pattern `<ip> - - [<_>] "<method> <path> <_>" <status> <size>`

# 추출된 필드로 필터
{app="nginx"} | pattern `<ip> - - [<_>] "<method> <path> <_>" <status> <size>` | status = "500"
```

### regexp 파싱

```logql
# 정규식으로 필드 추출
{app="my-app"} | regexp `(?P<level>\w+) (?P<message>.+)`
```

---

## 2. 라인 포맷 (Line Format)

파싱 후 로그 라인을 재구성합니다. Go template 문법을 사용합니다.

```logql
# 필요한 필드만 보이도록 재구성
{app="my-app"} | json | line_format "[{{.level}}] {{.message}}"

# 출력 예시
[error] DB connection failed
[info] Request completed
```

---

## 3. 메트릭 쿼리 (Metric Queries)

로그를 집계하여 숫자 시계열로 변환합니다. 대시보드 그래프와 알림에 사용합니다.

### count_over_time — 라인 수 카운트

```logql
# 최근 5분간 에러 로그 수
count_over_time({app="my-app"} |= "error" [5m])

# 1분간 전체 요청 수
count_over_time({app="my-app"}[1m])
```

### rate — 초당 비율

```logql
# 초당 에러 로그 발생률
rate({app="my-app"} |= "error" [5m])

# 네임스페이스별 초당 로그 발생률
rate({namespace="default"}[1m])
```

### bytes_over_time / bytes_rate — 바이트 기준

```logql
# 최근 5분간 로그 총 바이트 수
bytes_over_time({app="my-app"}[5m])

# 초당 로그 바이트 수
bytes_rate({app="my-app"}[5m])
```

---

## 4. 집계 (Aggregation)

메트릭 쿼리 결과를 레이블 기준으로 집계합니다.

### sum — 합계

```logql
# 앱별 초당 에러 발생률
sum by (app) (rate({namespace="default"} |= "error" [5m]))
```

### topk — 상위 N개

```logql
# 에러가 가장 많은 앱 Top 5
topk(5, sum by (app) (rate({namespace="default"} |= "error" [5m])))
```

### avg — 평균

```logql
# 앱별 평균 초당 로그 발생률
avg by (app) (rate({namespace="default"}[5m]))
```

---

## 5. Unwrap — 숫자 값 추출

로그에서 숫자 값을 추출하여 집계합니다.

```logql
# JSON 로그에서 duration_ms를 추출해 P99 계산
quantile_over_time(0.99,
  {app="my-app"}
  | json
  | unwrap duration_ms [5m]
) by (app)

# 평균 응답 시간
avg_over_time(
  {app="my-app"}
  | json
  | unwrap duration_ms [5m]
)
```

---

## 6. 복합 쿼리 예시

### 에러율 계산 (전체 요청 대비 에러 비율)

```logql
sum(rate({app="my-app"} | json | level = "error" [5m]))
/
sum(rate({app="my-app"} [5m]))
* 100
```

### 상태 코드별 요청 분포

```logql
sum by (status_code) (
  rate({app="my-app"} | json [5m])
)
```

### 슬로우 쿼리 탐지 (1초 이상 응답)

```logql
{app="my-app"} | json | duration_ms > 1000 | line_format "[SLOW] {{.method}} {{.path}} - {{.duration_ms}}ms"
```

---

## 참고

- [공식문서 - LogQL Metric Queries](https://grafana.com/docs/loki/latest/query/metric_queries/)
- [공식문서 - Log Parsers](https://grafana.com/docs/loki/latest/query/log_queries/#parser-expression)
- [공식문서 - Unwrap](https://grafana.com/docs/loki/latest/query/metric_queries/#unwrapped-range-aggregations)
