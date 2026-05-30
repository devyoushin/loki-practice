# loki-practice

EKS + Loki 3.x 기준으로 로그 수집, 레이블 설계, LogQL, Promtail, Grafana 연동, 알림, 보존 정책을 정리한 개인 학습 문서입니다.

## 빠른 시작

- 처음 볼 문서: `docs/install.md`
- 전체 흐름: 설치 -> 레이블/LogQL -> 수집 파이프라인 -> Grafana 연동 -> 알림/보존 -> 실습
- AI 작업 지침: `CLAUDE.md`

## 구조

```text
loki-practice/
├── README.md
├── CLAUDE.md
├── docs/
│   ├── README.md
│   ├── agents/
│   ├── rules/
│   ├── templates/
│   └── *.md
└── ops/
    ├── README.md
    └── config/     # Loki/Promtail values, 알림 규칙, 샘플 앱
```

## 학습 경로

| 단계 | 문서 |
|------|------|
| 설치 | `docs/install.md` |
| 핵심 개념 | `docs/label-guide.md`, `docs/logql-basics-guide.md` |
| 쿼리 | `docs/logql-advanced-guide.md`, `docs/log-query-test.md` |
| 수집 | `docs/promtail-guide.md` |
| 시각화 | `docs/grafana-integration-guide.md` |
| 운영 | `docs/alerting-guide.md`, `docs/retention-guide.md` |

## 환경

| 항목 | 값 |
|------|-----|
| Platform | EKS |
| Loki | 3.x |
| Collector | Promtail 3.x |
| Namespace | `monitoring` |
| App namespace | `default` |
