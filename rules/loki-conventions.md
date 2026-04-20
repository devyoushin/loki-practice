# Loki 컨벤션 규칙

## 레이블 컨벤션
- 표준 레이블: `namespace`, `pod`, `container`, `node`, `job`, `app`
- 레이블명: snake_case
- 고카디널리티 값 레이블 사용 금지: user_id, request_id, trace_id, IP
- 레이블 수: 스트림당 최대 15개 이하
- 환경 레이블 필수: `env` (production, staging, development)

## 스트림 셀렉터 규칙
```logql
# 올바른 예 — 좁은 범위 선택
{namespace="production", app="payment-service"}

# 잘못된 예 — 너무 넓은 범위
{namespace=~".+"}
```

## 로그 레벨 파싱
```logql
# JSON 구조 로그
{namespace="production"} | json | level="error"

# 구조화되지 않은 로그
{namespace="production"} |= "ERROR" != "DEBUG"
```

## 알림 규칙 (Loki Rules)
```yaml
groups:
  - name: loki-alerts
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate({namespace="production"} |= "ERROR" [5m])) by (app)
          /
          sum(rate({namespace="production"} [5m])) by (app)
          > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "{{ $labels.app }} 에러율 5% 초과"
          runbook_url: "..."
```

## 보존 정책
- 프로덕션 로그: 30일
- 스테이징 로그: 7일
- 개발 로그: 3일
- 감사(Audit) 로그: 1년 (별도 스트림)

## Promtail/Alloy 파이프라인 규칙
- 수집 전 레이블 정규화 (relabel_configs)
- 멀티라인 로그 처리 (multiline stage)
- 민감 정보 마스킹 (replace stage)
