# Loki 보존 정책 및 스토리지 가이드

Loki의 스토리지 구조와 로그 보존 기간 설정을 다룹니다.

---

## Loki 스토리지 구조

```
[Loki]
  │
  ├── Index (인덱스)        — 레이블 → 청크 위치 매핑
  │     S3 / DynamoDB / BoltDB 등
  │
  └── Chunks (청크)         — 실제 로그 데이터 (압축)
        S3 / GCS / 로컬 파일시스템 등
```

> Loki는 로그를 청크로 묶어 오브젝트 스토리지(S3 등)에 저장하기 때문에 Elasticsearch보다 훨씬 저렴합니다.

---

## 1. 스토리지 모드

### Filesystem (개발/테스트용)

```yaml
# loki-values.yaml
loki:
  storage:
    type: filesystem
  commonConfig:
    path_prefix: /var/loki
```

### S3 (EKS 운영 환경 권장)

```yaml
# loki-values.yaml
loki:
  storage:
    type: s3
    s3:
      region: ap-northeast-2
      bucketnames: my-loki-bucket
      s3ForcePathStyle: false

serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/loki-s3-role
```

S3 IAM 정책 (최소 권한):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-loki-bucket",
        "arn:aws:s3:::my-loki-bucket/*"
      ]
    }
  ]
}
```

---

## 2. 보존 기간 설정

### 전역 보존 기간

```yaml
# loki-values.yaml
loki:
  limits_config:
    retention_period: 30d    # 전체 로그 30일 보존

  compactor:
    working_directory: /var/loki/compactor
    retention_enabled: true
    delete_request_store: filesystem
```

### 네임스페이스/스트림별 보존 기간 (Per-tenant)

```yaml
loki:
  limits_config:
    retention_period: 30d    # 기본값

  # 특정 tenant(namespace)만 다른 보존 기간 설정
  per_tenant_override_config: /etc/loki/overrides.yaml
```

```yaml
# overrides.yaml
overrides:
  production:
    retention_period: 90d    # production 로그는 90일 보존
  development:
    retention_period: 7d     # dev 로그는 7일만 보존
```

---

## 3. Compactor

Compactor는 오래된 청크를 정리하고 보존 정책을 실행합니다.

```yaml
loki:
  compactor:
    working_directory: /var/loki/compactor
    retention_enabled: true           # 보존 정책 적용 활성화
    retention_delete_delay: 2h        # 만료 후 실제 삭제까지 유예 시간
    retention_delete_worker_count: 150
    compaction_interval: 10m          # 압축 실행 주기
```

---

## 4. 인덱스 설정 (TSDB 권장)

Loki 3.x에서는 TSDB 인덱스가 기본이자 권장 설정입니다.

```yaml
loki:
  schemaConfig:
    configs:
      - from: "2024-01-01"
        store: tsdb
        object_store: s3
        schema: v13
        index:
          prefix: loki_index_
          period: 24h
```

---

## 5. 스토리지 사용량 모니터링

```bash
# 로컬 파일시스템 사용량 확인
kubectl exec -n monitoring loki-0 -- du -sh /var/loki/

# S3 버킷 사용량 (AWS CLI)
aws s3 ls s3://my-loki-bucket --recursive --human-readable --summarize

# Loki 메트릭으로 확인
kubectl port-forward svc/loki -n monitoring 3100:3100
curl http://localhost:3100/metrics | grep loki_ingester_chunk
```

### 주요 메트릭

| 메트릭 | 설명 |
|--------|------|
| `loki_ingester_chunks_stored_total` | 저장된 청크 총 수 |
| `loki_boltdb_shipper_chunks_downloaded_total` | 다운로드된 청크 수 |
| `loki_compactor_oldest_processed_timestamp_seconds` | 마지막 compaction 시각 |

---

## 참고

- [공식문서 - Storage](https://grafana.com/docs/loki/latest/storage/)
- [공식문서 - Retention](https://grafana.com/docs/loki/latest/operations/storage/retention/)
- [공식문서 - S3 설정](https://grafana.com/docs/loki/latest/storage/#aws-s3)
