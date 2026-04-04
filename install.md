# Helm으로 Loki 스택 설치

> Loki + Promtail + Grafana를 EKS에 설치합니다.

### 1. 사전 준비: Helm 저장소 및 네임스페이스
---
레포지토리 추가 및 업데이트
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

네임스페이스 생성
```bash
kubectl create namespace monitoring
```

### 2. Loki 설치 (SingleBinary 모드)
---
소규모 환경에 적합한 단일 바이너리 모드로 설치합니다.

```bash
helm install loki grafana/loki \
  --namespace monitoring \
  -f loki/loki-values.yaml
```

설치 확인
```bash
kubectl get pods -n monitoring
# loki-0 가 Running 상태인지 확인
```

### 3. Promtail 설치
---
Promtail은 각 노드에서 로그를 수집하는 DaemonSet으로 동작합니다.

```bash
helm install promtail grafana/promtail \
  --namespace monitoring \
  -f loki/promtail-values.yaml
```

설치 확인
```bash
# DaemonSet이 모든 노드에 배포되었는지 확인
kubectl get daemonset -n monitoring
kubectl get pods -n monitoring -l app.kubernetes.io/name=promtail
```

### 4. Grafana 설치
---
```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set adminPassword=admin \
  --set datasources."datasources\.yaml".apiVersion=1 \
  --set datasources."datasources\.yaml".datasources[0].name=Loki \
  --set datasources."datasources\.yaml".datasources[0].type=loki \
  --set datasources."datasources\.yaml".datasources[0].url=http://loki.monitoring:3100 \
  --set datasources."datasources\.yaml".datasources[0].access=proxy \
  --set datasources."datasources\.yaml".datasources[0].isDefault=true
```

### 5. Grafana 접속
---
```bash
# 포트 포워딩
kubectl port-forward svc/grafana -n monitoring 3000:80

# 브라우저에서 http://localhost:3000 접속 (admin / admin)
```

### 6. 전체 설치 상태 확인
---
```bash
kubectl get all -n monitoring
```

예상 출력:
```
NAME                           READY   STATUS    RESTARTS
pod/grafana-xxx                1/1     Running   0
pod/loki-0                     1/1     Running   0
pod/promtail-xxxxx (노드별)    1/1     Running   0

NAME                        TYPE        CLUSTER-IP
service/grafana             ClusterIP   10.100.x.x
service/loki                ClusterIP   10.100.x.x
service/loki-headless       ClusterIP   None

NAME                      DESIRED   CURRENT   READY
daemonset.apps/promtail   3         3         3
```
