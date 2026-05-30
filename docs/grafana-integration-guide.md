# Grafana + Loki 연동 가이드

Grafana에서 Loki 데이터소스를 설정하고 로그를 조회, 시각화하는 방법을 다룹니다.

---

## 1. Loki 데이터소스 등록

```
Grafana → Configuration → Data Sources → Add data source → Loki
```

| 설정 | 값 | 설명 |
|------|-----|------|
| URL | `http://loki.monitoring:3100` | Loki 서비스 주소 |
| HTTP Method | GET | 기본값 |
| Max lines | 1000 | 한 번에 가져올 최대 로그 줄 수 |

```bash
# 저장 후 "Test & Save" 클릭
# "Data source connected and labels found" 메시지 확인
```

---

## 2. Explore로 로그 조회

Explore는 로그를 즉석에서 탐색할 때 사용합니다.

```
Grafana → Explore → 데이터소스: Loki 선택
```

### Label Browser 사용

```
1. "Log browser" 클릭
2. 레이블 선택 (namespace, app 등)
3. "Show logs" 클릭
```

### LogQL 직접 입력

```logql
{namespace="default", app="my-app"} |= "error" | json | level = "error"
```

### 유용한 Explore 기능

- **Live tail**: 실시간 로그 스트리밍 (`Live` 버튼)
- **Log context**: 특정 로그 라인 전후 맥락 확인 (로그 라인 옆 `Context` 버튼)
- **Split view**: 두 쿼리 결과를 나란히 비교

---

## 3. 대시보드에 로그 패널 추가

### Logs 패널

```
Dashboard → Add panel → Visualization: Logs
→ Query: {namespace="default", app="my-app"} |= "error"
→ Options:
    - Deduplication: Exact (중복 제거)
    - Order: Newest first
    - Wrap lines: On
```

### Time series 패널 (에러 추이)

```
Dashboard → Add panel → Visualization: Time series
→ Query: sum(rate({namespace="default"} |= "error" [5m])) by (app)
→ 에러 발생률 그래프로 표시
```

### Stat 패널 (에러 건수)

```
Dashboard → Add panel → Visualization: Stat
→ Query: sum(count_over_time({namespace="default"} |= "error" [1h]))
→ 최근 1시간 에러 총 건수 표시
```

---

## 4. 공식 Loki 대시보드 임포트

```
Grafana → Dashboards → Import → ID 입력
```

| 대시보드 | ID | 내용 |
|----------|----|------|
| Loki Dashboard quick search | 12611 | 로그 검색 UI |
| Loki / Promtail | 10880 | Loki + Promtail 운영 메트릭 |
| Kubernetes Logs (Loki) | 15141 | 쿠버네티스 전체 로그 뷰 |

---

## 5. 변수(Variables)로 대시보드 동적 필터링

대시보드에 드롭다운 필터를 추가합니다.

```
Dashboard Settings → Variables → Add variable

Type: Query
Data source: Loki
Query: label_values(namespace)       # namespace 레이블의 모든 값 목록
Name: namespace
Label: Namespace
```

대시보드 쿼리에서 변수 사용:
```logql
{namespace="$namespace", app="$app"} |= "$search"
```

---

## 6. Prometheus + Loki 연동 (Exemplar)

Prometheus 메트릭에서 Loki 로그로 바로 이동합니다.

```yaml
# Grafana 데이터소스 설정 (Prometheus)
# Derived fields 설정:
Name: TraceID
Regex: traceID=(\w+)
URL: http://jaeger:16686/trace/$${__value.raw}

# Loki 데이터소스에서도 동일하게 설정 가능
```

---

## 7. Grafana Explore에서 Loki ↔ Tempo 연동

분산 트레이싱과 로그를 함께 봅니다.

```
Grafana → Explore → Split
→ 왼쪽: Loki 로그에서 trace_id 클릭
→ 오른쪽: Tempo에서 해당 trace 자동 표시
```

---

## 참고

- [공식문서 - Grafana Loki Integration](https://grafana.com/docs/loki/latest/visualize/grafana/)
- [공식문서 - Explore](https://grafana.com/docs/grafana/latest/explore/)
- [Grafana 대시보드 검색](https://grafana.com/grafana/dashboards/?search=loki)
