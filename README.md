# loki-practice

EKS 환경에서 Loki 3.x 기반 로그 수집, 레이블 설계, LogQL, Promtail, Grafana 연동, 알림, 보존 정책을 학습하는 문서 저장소입니다.

## 먼저 볼 문서

| 목적 | 문서 |
|------|------|
| 전체 문서 목차 보기 | [docs/README.md](docs/README.md) |
| Loki 설치하기 | [docs/install/install.md](docs/install/install.md) |
| 레이블 설계 이해하기 | [docs/concepts/label-guide.md](docs/concepts/label-guide.md) |
| LogQL 배우기 | [docs/logql/logql-basics-guide.md](docs/logql/logql-basics-guide.md), [docs/logql/logql-advanced-guide.md](docs/logql/logql-advanced-guide.md) |
| 로그 수집 파이프라인 구성하기 | [docs/collection/promtail-guide.md](docs/collection/promtail-guide.md) |
| Grafana와 연동하기 | [docs/integrations/grafana-integration-guide.md](docs/integrations/grafana-integration-guide.md) |
| 쿼리 실습하기 | [docs/tutorials/log-query-test.md](docs/tutorials/log-query-test.md) |
| 운영 YAML 확인하기 | [ops/README.md](ops/README.md) |

## 추천 학습 순서

1. [Loki 설치](docs/install/install.md)
2. [레이블 설계](docs/concepts/label-guide.md)
3. [LogQL 기초](docs/logql/logql-basics-guide.md)
4. [LogQL 심화](docs/logql/logql-advanced-guide.md)
5. [Promtail 수집 파이프라인](docs/collection/promtail-guide.md)
6. [Grafana 연동](docs/integrations/grafana-integration-guide.md)
7. [알림](docs/operations/alerting-guide.md), [보존 정책](docs/operations/retention-guide.md)
8. [로그 쿼리 실습](docs/tutorials/log-query-test.md)

## 디렉터리 구조

```text
loki-practice/
├── README.md
├── CLAUDE.md          # AI 작업 지침
├── docs/
│   ├── README.md     # 문서 전체 목차
│   ├── install/      # 설치와 업그레이드
│   ├── concepts/     # 레이블 설계
│   ├── logql/        # LogQL 기초/심화
│   ├── collection/   # Promtail 수집 파이프라인
│   ├── integrations/ # Grafana 연동
│   ├── operations/   # 알림과 보존 정책
│   ├── tutorials/    # 쿼리 실습
│   ├── agents/       # AI 역할별 작업 지침
│   ├── rules/        # 문서/운영 규칙
│   └── templates/    # 서비스 문서, 런북, 장애 보고서 템플릿
└── ops/
    ├── README.md
    └── config/       # Loki/Promtail values, 알림 규칙, 샘플 앱
```

## 문서 분류

| 분류 | 내용 |
|------|------|
| 설치 | Helm, systemd, Docker Compose, 업그레이드 |
| 핵심 개념 | 레이블 설계, 카디널리티 관리 |
| 쿼리 | LogQL 기초, 파싱, 메트릭 쿼리, 집계 |
| 수집/연동 | Promtail, Grafana Explore, 대시보드 |
| 운영/실습 | 알림, 보존 정책, 로그 쿼리 검증 |

## 환경

| 항목 | 값 |
|------|-----|
| Platform | EKS |
| Loki | 3.x |
| Collector | Promtail 3.x |
| Namespace | `monitoring` |
| App namespace | `default` |
