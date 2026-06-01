# Loki 설치 (Helm)

EKS 환경에서 Loki, Promtail, Grafana를 Helm으로 설치한다.

## 사전 준비

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
```

## 설치

```bash
helm install loki grafana/loki \
  --namespace monitoring \
  -f ../ops/config/loki/loki-values.yaml

helm install promtail grafana/promtail \
  --namespace monitoring \
  -f ../ops/config/loki/promtail-values.yaml
```

## Grafana 연동

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set adminPassword=admin \
  --set datasources."datasources\.yaml".datasources[0].name=Loki \
  --set datasources."datasources\.yaml".datasources[0].type=loki \
  --set datasources."datasources\.yaml".datasources[0].url=http://loki.monitoring:3100
```

## 확인

```bash
kubectl get pods -n monitoring
kubectl get daemonset -n monitoring
kubectl port-forward svc/grafana -n monitoring 3000:80
```

## 업그레이드 / 삭제

```bash
helm upgrade loki grafana/loki -n monitoring -f ../ops/config/loki/loki-values.yaml
helm upgrade promtail grafana/promtail -n monitoring -f ../ops/config/loki/promtail-values.yaml
helm uninstall loki -n monitoring
helm uninstall promtail -n monitoring
helm uninstall grafana -n monitoring
```
