# Loki 레이블 설계 가이드

Loki는 로그 내용(전문)을 인덱싱하지 않고, **레이블만 인덱싱**합니다.
따라서 레이블 설계가 성능과 비용에 직접적인 영향을 줍니다.

---

## 레이블이란?

레이블은 로그 스트림을 식별하는 키-값 쌍입니다.

```
{namespace="default", app="my-app", env="production"}
```

동일한 레이블 조합 = 동일한 로그 스트림 → 하나의 청크(chunk)에 저장됩니다.

---

## 레이블 카디널리티 (Cardinality)

**카디널리티** = 특정 레이블이 가질 수 있는 고유한 값의 수

### 낮은 카디널리티 (권장)

```
namespace: "default", "monitoring", "production"  → 값 3개
env: "dev", "staging", "prod"                      → 값 3개
```

→ 스트림 수가 적어 Loki 성능이 좋음

### 높은 카디널리티 (피해야 함)

```
pod: "my-app-abc123", "my-app-def456", "my-app-ghi789", ...
user_id: "1001", "1002", "1003", ... (수백만 가능)
trace_id: "a1b2c3...", "d4e5f6..."
```

→ 스트림 수가 폭발적으로 증가 → 메모리/인덱스 증가 → 성능 저하

> **규칙**: 레이블 값이 동적으로 늘어나는 것들(trace_id, user_id, request_id)은 절대 레이블로 쓰지 말고, 로그 내용에 포함시키세요.

---

## 정적 레이블 vs 동적 레이블

### 정적 레이블 (Static Labels) — 레이블로 사용 권장

수집 시점에 확정되는 메타데이터

```yaml
# Promtail이 자동으로 붙여주는 정적 레이블
namespace: default
pod: my-app-abc123
container: my-app
node_name: ip-10-0-1-1
```

```yaml
# 직접 추가하는 정적 레이블
env: production
team: backend
service: payment
```

### 동적 레이블 (Dynamic Labels) — 레이블로 사용 금지

로그 내용에서 파싱되는 값 (매 요청마다 달라짐)

```
# 잘못된 예시 — 레이블로 절대 쓰면 안 됨
user_id: "1001"
request_id: "abc-123-def"
trace_id: "a1b2c3d4"
```

이 값들은 로그 내용에 포함하고, 쿼리 시 `| json | user_id = "1001"` 처럼 파싱해서 필터링하세요.

---

## 좋은 레이블 설계 예시

```
# 좋음: 카디널리티 낮고, 쿼리에 자주 사용
{namespace="default", app="my-app", env="production"}

# 나쁨: pod 이름은 자동 확장/축소로 계속 바뀜
{namespace="default", pod="my-app-abc123xyz"}
```

---

## 레이블 수 제한

Loki는 기본적으로 스트림당 레이블 수에 제한이 있습니다.

```yaml
# loki-values.yaml
limits_config:
  max_label_names_per_series: 15    # 스트림당 최대 레이블 수 (기본: 15)
  max_label_value_length: 2048      # 레이블 값 최대 길이
```

레이블이 많을수록 카디널리티가 곱셈으로 증가합니다.

```
namespace (3가지) × app (10가지) × env (3가지) = 90개 스트림
namespace (3가지) × app (10가지) × env (3가지) × pod (100가지) = 9000개 스트림
```

---

## 레이블 상태 확인

```bash
# Loki가 현재 관리 중인 스트림 수 확인
curl http://localhost:3100/metrics | grep loki_ingester_streams_created_total

# 레이블 이름 목록 조회 (Grafana Explore에서)
# label_names()

# 특정 레이블의 값 목록 조회
# label_values(app)
```

---

## 참고

- [공식문서 - Labels](https://grafana.com/docs/loki/latest/get-started/labels/)
- [공식문서 - Cardinality](https://grafana.com/docs/loki/latest/best-practices/)
