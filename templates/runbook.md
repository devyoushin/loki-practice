# [알림명] — Loki 런북

> 심각도: Critical / Warning / Info
> 담당팀: | 최종 수정: YYYY-MM-DD

## 알림 요약
<!-- 이 알림이 무엇을 감지하는지 한 문장으로 -->

## 알림 조건

```logql
# 알림 트리거 LogQL
```

## 영향도
<!-- 이 알림이 발생했을 때 사용자/서비스에 미치는 영향 -->

## 즉시 확인 사항

```bash
# 1. Loki 컴포넌트 상태 확인
kubectl get pods -n monitoring -l app=loki

# 2. logcli로 로그 직접 확인
logcli query '{namespace="production"}' --limit=50 --since=10m

# 3. Loki 스토리지 상태
kubectl exec -n monitoring loki-0 -- df -h /var/loki
```

## 진단 단계

### 1단계: 수집 상태 확인
```bash
# 수집기 상태 확인
kubectl get pods -n monitoring -l app=promtail
```

### 2단계: 원인 분석
<!-- 일반적인 원인 목록 -->

### 3단계: 해결 조치
<!-- 단계별 해결 방법 -->

## 에스컬레이션
- 15분 내 해결 불가 시: 팀 리드 호출
- 로그 수집 전면 중단 시: 인시던트 선언

## 참고
- 관련 Grafana 대시보드:
- 관련 문서:
