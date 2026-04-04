# LogQL 기초 가이드

LogQL은 Loki의 쿼리 언어입니다. PromQL(Prometheus)에서 영감을 받아 설계되었으며, 두 가지 쿼리 타입을 지원합니다.

| 타입 | 반환값 | 예시 |
|------|--------|------|
| **Log Query** | 로그 라인 목록 | Grafana Explore, 로그 확인 |
| **Metric Query** | 숫자 (시계열) | 대시보드 그래프, 알림 |

---

## 1. 로그 스트림 선택자 (Log Stream Selector)

모든 LogQL 쿼리는 `{}` 로 시작합니다. 레이블로 로그 스트림을 선택합니다.

```logql
# namespace가 default인 모든 로그
{namespace="default"}

# namespace=default 이고 app=my-app인 로그
{namespace="default", app="my-app"}

# 여러 값 중 하나 (정규식)
{namespace=~"default|production"}

# 특정 값 제외
{namespace="default", app!="prometheus"}
```

### 레이블 매처 종류

| 연산자 | 의미 | 예시 |
|--------|------|------|
| `=` | 정확히 일치 | `app="my-app"` |
| `!=` | 불일치 | `app!="prometheus"` |
| `=~` | 정규식 일치 | `app=~"my-app.*"` |
| `!~` | 정규식 불일치 | `app!~"kube.*"` |

> **중요**: 스트림 선택자에 반드시 하나 이상의 레이블이 있어야 합니다. `{}` 단독 사용은 불가합니다.

---

## 2. 라인 필터 (Line Filter)

스트림 선택자로 가져온 로그에서 특정 문자열이 있는 라인만 필터링합니다.

```logql
# "error" 문자열 포함 (대소문자 구분)
{app="my-app"} |= "error"

# "debug" 문자열 제외
{app="my-app"} != "debug"

# 정규식 포함
{app="my-app"} |~ "timeout|connection refused"

# 정규식 제외
{app="my-app"} !~ "health.*check"
```

### 파이프라인으로 연결

여러 필터를 파이프(`|`)로 연결할 수 있습니다.

```logql
{namespace="default", app="my-app"}
  |= "error"
  != "404"
  |~ "user_id=\d+"
```

---

## 3. 레이블 필터 (Label Filter)

파싱 후 추출된 레이블을 기준으로 필터링합니다.

```logql
# 파싱 후 status_code가 500인 라인만
{app="my-app"} | json | status_code = 500

# 문자열 비교
{app="my-app"} | json | level = "error"

# 숫자 비교
{app="my-app"} | json | duration > 1000
```

### 레이블 필터 연산자

| 연산자 | 예시 |
|--------|------|
| `=` | `level = "error"` |
| `!=` | `level != "info"` |
| `>` | `duration > 500` |
| `>=` | `status >= 400` |
| `<` | `size < 1024` |
| `=~` | `path =~ "/api/.*"` |

---

## 4. 자주 쓰는 쿼리 예시

### 특정 앱의 에러 로그
```logql
{namespace="default", app="my-app"} |= "ERROR"
```

### 최근 5분간 특정 Pod의 로그
```logql
{namespace="default", pod=~"my-app-.*"}
```

### 특정 HTTP 상태코드 로그 (JSON 형식)
```logql
{app="my-app"} | json | status >= 500
```

### 로그 라인 수 확인 (메트릭 쿼리)
```logql
count_over_time({app="my-app"}[5m])
```

---

## 5. 기본 Kubernetes 레이블

Promtail이 자동으로 붙여주는 레이블 목록입니다.

| 레이블 | 예시 값 | 설명 |
|--------|---------|------|
| `namespace` | `default` | Pod 네임스페이스 |
| `pod` | `my-app-abc123` | Pod 이름 |
| `container` | `my-app` | 컨테이너 이름 |
| `node_name` | `ip-10-0-1-1` | 노드 이름 |
| `app` | `my-app` | `app` 레이블 값 |
| `stream` | `stdout` | 표준 출력 구분 |

---

## 참고

- [공식문서 - LogQL](https://grafana.com/docs/loki/latest/query/)
- [공식문서 - Log Stream Selector](https://grafana.com/docs/loki/latest/query/log_queries/#log-stream-selector)
