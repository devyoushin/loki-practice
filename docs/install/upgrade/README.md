# Loki 업그레이드 가이드

Loki 업그레이드는 Loki, Promtail, Grafana datasource와 저장소 schema 설정에 영향을 줄 수 있습니다. schema 변경, index period, object storage 설정 변경은 릴리즈 노트를 먼저 확인합니다.

## 1. 사전 점검

```bash
export NAMESPACE="monitoring"
export LOKI_RELEASE="loki"
export PROMTAIL_RELEASE="promtail"
export LOKI_VALUES="../../../ops/config/loki/loki-values.yaml"
export PROMTAIL_VALUES="../../../ops/config/loki/promtail-values.yaml"

helm status ${LOKI_RELEASE} -n ${NAMESPACE}
helm history ${LOKI_RELEASE} -n ${NAMESPACE}
helm get values ${LOKI_RELEASE} -n ${NAMESPACE} > loki-values-before-upgrade.yaml
kubectl get pods,svc,pvc -n ${NAMESPACE}
```

## 2. Helm 업그레이드

```bash
helm repo update grafana

helm upgrade ${LOKI_RELEASE} grafana/loki \
  --namespace ${NAMESPACE} \
  --values ${LOKI_VALUES} \
  --timeout 10m \
  --wait

helm upgrade ${PROMTAIL_RELEASE} grafana/promtail \
  --namespace ${NAMESPACE} \
  --values ${PROMTAIL_VALUES} \
  --timeout 10m \
  --wait
```

## 3. 확인

```bash
kubectl get pods -n ${NAMESPACE}
kubectl rollout status daemonset/${PROMTAIL_RELEASE} -n ${NAMESPACE}
kubectl logs -n ${NAMESPACE} -l app.kubernetes.io/name=loki --tail=100
kubectl logs -n ${NAMESPACE} -l app.kubernetes.io/name=promtail --tail=100
```

Grafana Explore에서 최근 로그가 조회되는지, Promtail target이 정상인지 확인합니다.

## 4. 롤백

```bash
helm history ${LOKI_RELEASE} -n ${NAMESPACE}
helm rollback ${LOKI_RELEASE} <REVISION> -n ${NAMESPACE} --wait

helm history ${PROMTAIL_RELEASE} -n ${NAMESPACE}
helm rollback ${PROMTAIL_RELEASE} <REVISION> -n ${NAMESPACE} --wait
```

저장소 schema 변경 후에는 이전 Loki가 새 index/chunk 형식을 읽지 못할 수 있습니다. major 업그레이드는 staging에서 쿼리와 ingestion을 먼저 검증합니다.

## 5. systemd / Docker Compose

systemd 설치는 Loki와 Promtail 바이너리를 순차 교체하고 서비스를 재시작합니다. Docker Compose 설치는 image tag를 올린 뒤 `docker compose pull && docker compose up -d`를 실행합니다. 설정 파일과 로컬 데이터 디렉터리를 먼저 백업합니다.

