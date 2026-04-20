# 모니터링 규칙 — loki-practice

## Loki 자체 모니터링

### 핵심 메트릭

```promql
# 수집 속도 (라인/초)
sum(rate(loki_distributor_lines_received_total[5m]))

# Ingester 청크 수
sum(loki_ingester_chunks_stored_total)

# 쿼리 지연 (p99)
histogram_quantile(0.99, rate(loki_request_duration_seconds_bucket{route=~"loki_api.*query.*"}[5m]))

# 스토리지 사용량
loki_boltdb_shipper_compact_tables_operation_total

# 수집 오류율
rate(loki_distributor_lines_received_total[5m]) - rate(loki_distributor_bytes_received_total[5m])
```

### 알림 규칙 (권장)

| 알림 | 조건 | 심각도 |
|------|------|--------|
| LokiIngesterDown | `up{job="loki"} == 0` for 1m | critical |
| LokiQuerySlow | `histogram_quantile(0.99, ...) > 30` | warning |
| LokiStorageFull | 디스크 90% 초과 | critical |
| LokiHighCardinality | 스트림 수 급증 | warning |

## SLO 정의 (Loki 서비스)

| SLI | 목표 | 측정 방법 |
|-----|------|---------|
| 수집 가용성 | 99.9% | 수집 오류율 < 0.1% |
| 쿼리 p99 지연 | < 30초 | `loki_request_duration_seconds` |
| 로그 신선도 | < 30초 | Ingester flush 지연 |

## 대시보드 구성 (Grafana)
- **Overview**: 수집 속도, 쿼리 수, 오류율
- **Ingester**: 청크 상태, 메모리 사용량
- **Querier**: 쿼리 지연, 동시 쿼리 수
- **Storage**: S3 업로드 상태, Compactor 실행 현황
