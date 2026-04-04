# Promtail 가이드

Promtail은 Loki의 공식 로그 수집 에이전트입니다. 각 노드에 DaemonSet으로 배포되어 Pod의 로그를 수집하고 Loki로 전송합니다.

---

## 동작 원리

```
[Node]
  ├── /var/log/pods/default_my-app-xxx/my-app/0.log
  ├── /var/log/pods/default_nginx-xxx/nginx/0.log
  └── ...
       │
       ▼
  [Promtail DaemonSet]
       │  1. 파일 디스커버리 (kubernetes_sd_configs)
       │  2. 레이블 재매핑 (relabel_configs)
       │  3. Pipeline stages (파싱, 레이블 추출, 필터)
       │
       ▼
  [Loki HTTP Push API]
```

---

## 1. 기본 설정 구조

```yaml
# promtail-values.yaml
config:
  clients:
    - url: http://loki.monitoring:3100/loki/api/v1/push

  scrape_configs:
    - job_name: kubernetes-pods
      kubernetes_sd_configs:
        - role: pod
      pipeline_stages:
        - cri: {}          # CRI(Container Runtime Interface) 형식 파싱
      relabel_configs:
        # 필요한 레이블만 유지
        - source_labels: [__meta_kubernetes_namespace]
          target_label: namespace
        - source_labels: [__meta_kubernetes_pod_name]
          target_label: pod
        - source_labels: [__meta_kubernetes_pod_container_name]
          target_label: container
        - source_labels: [__meta_kubernetes_pod_label_app]
          target_label: app
```

---

## 2. Pipeline Stages

Pipeline stages는 로그가 Loki로 전송되기 전에 변환/필터링합니다.

### cri — CRI 로그 형식 파싱

Kubernetes의 표준 로그 형식(`timestamp stream flags message`)을 파싱합니다.

```yaml
pipeline_stages:
  - cri: {}
```

### json — JSON 로그 파싱

```yaml
pipeline_stages:
  - cri: {}
  - json:
      expressions:
        level: level          # JSON 필드 "level" → 임시 레이블 "level"
        app_name: app         # JSON 필드 "app" → 임시 레이블 "app_name"
```

### labels — 임시 레이블을 실제 레이블로 승격

```yaml
pipeline_stages:
  - cri: {}
  - json:
      expressions:
        level: level
  - labels:
      level:       # "level" 임시 레이블을 Loki 스트림 레이블로 등록
```

> **주의**: `labels` stage에 너무 많은 필드를 넣으면 카디널리티 문제가 생깁니다. 카디널리티가 낮은 `level` 같은 필드만 레이블로 승격하세요.

### match — 조건에 맞는 로그만 처리

```yaml
pipeline_stages:
  - cri: {}
  - match:
      selector: '{app="my-app"}'
      stages:
        - json:
            expressions:
              level: level
        - labels:
            level:
```

### drop — 특정 로그 제외 (비용 절감)

```yaml
pipeline_stages:
  - cri: {}
  - drop:
      expression: ".*health.*check.*"    # 헬스체크 로그 제외
  - drop:
      source: level
      value: "debug"                     # debug 레벨 로그 제외
```

---

## 3. 멀티라인 로그 처리

Java 스택 트레이스처럼 여러 줄에 걸친 로그를 하나의 로그 이벤트로 묶습니다.

```yaml
pipeline_stages:
  - cri: {}
  - multiline:
      firstline: '^\d{4}-\d{2}-\d{2}'   # 날짜로 시작하는 줄이 새 로그의 시작
      max_wait_time: 3s
```

예시:
```
# 이 세 줄이 하나의 로그 이벤트로 묶임
2024-01-10 ERROR DB connection failed
  at com.example.DBClient.connect(DBClient.java:42)
  at com.example.UserService.getUser(UserService.java:18)
```

---

## 4. 특정 네임스페이스/앱만 수집

```yaml
relabel_configs:
  # monitoring 네임스페이스의 로그는 수집 제외
  - source_labels: [__meta_kubernetes_namespace]
    action: drop
    regex: monitoring

  # app 레이블이 없는 Pod는 수집 제외
  - source_labels: [__meta_kubernetes_pod_label_app]
    action: drop
    regex: ^$
```

---

## 5. 상태 및 디버깅

```bash
# Promtail Pod 로그 확인
kubectl logs -n monitoring -l app.kubernetes.io/name=promtail --tail=50

# Promtail 타겟 상태 확인 (수집 중인 파일 목록)
kubectl port-forward -n monitoring daemonset/promtail 9080:9080
# 브라우저에서 http://localhost:9080/targets 접속

# Promtail 설정 확인
kubectl get configmap -n monitoring promtail -o yaml
```

### 자주 발생하는 문제

| 증상 | 원인 | 해결 |
|------|------|------|
| 로그가 Loki에 안 보임 | Promtail → Loki 연결 실패 | `clients.url` 확인, Loki 서비스 이름 확인 |
| 특정 Pod 로그 누락 | relabel_configs에서 drop됨 | `/targets` 페이지에서 타겟 상태 확인 |
| 레이블 없음 | relabel_configs 누락 | `__meta_kubernetes_*` 레이블 매핑 확인 |
| 멀티라인 분리됨 | multiline stage 미설정 | `firstline` 정규식을 로그 형식에 맞게 설정 |

---

## 참고

- [공식문서 - Promtail](https://grafana.com/docs/loki/latest/send-data/promtail/)
- [공식문서 - Pipeline Stages](https://grafana.com/docs/loki/latest/send-data/promtail/stages/)
- [공식문서 - relabel_configs](https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#relabel_configs)
