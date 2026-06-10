# Loki Docs

Loki를 처음 보는 사람이 설치, 레이블 설계, 로그 수집, LogQL, Grafana 연동, 운영 정책, 실습까지 순서대로 따라갈 수 있도록 정리한 문서 디렉터리입니다.

## 빠른 길잡이

| 지금 하고 싶은 일 | 열 문서 |
|------|------|
| Loki 설치 방식을 고르기 | [install/install.md](install/install.md) |
| Loki 업그레이드 또는 롤백하기 | [install/upgrade/README.md](install/upgrade/README.md) |
| 레이블과 카디널리티 이해하기 | [concepts/label-guide.md](concepts/label-guide.md) |
| LogQL 기본 문법 보기 | [logql/logql-basics-guide.md](logql/logql-basics-guide.md) |
| LogQL 파싱/집계/메트릭 쿼리 보기 | [logql/logql-advanced-guide.md](logql/logql-advanced-guide.md) |
| Promtail 수집 파이프라인 설정하기 | [collection/promtail-guide.md](collection/promtail-guide.md) |
| Grafana Explore와 대시보드 연결하기 | [integrations/grafana-integration-guide.md](integrations/grafana-integration-guide.md) |
| 알림 규칙 만들기 | [operations/alerting-guide.md](operations/alerting-guide.md) |
| 보존 정책과 스토리지 이해하기 | [operations/retention-guide.md](operations/retention-guide.md) |
| 로그 쿼리를 직접 실습하기 | [tutorials/log-query-test.md](tutorials/log-query-test.md) |

## 추천 읽기 순서

| 순서 | 문서 | 핵심 내용 |
|------|------|------|
| 1 | [install/install.md](install/install.md) | Helm, systemd, Docker Compose 설치 방식 |
| 2 | [concepts/label-guide.md](concepts/label-guide.md) | 레이블 설계와 카디널리티 관리 |
| 3 | [logql/logql-basics-guide.md](logql/logql-basics-guide.md) | 스트림 선택자, 라인 필터, 레이블 필터 |
| 4 | [logql/logql-advanced-guide.md](logql/logql-advanced-guide.md) | 파싱, 집계, metric query, unwrap |
| 5 | [collection/promtail-guide.md](collection/promtail-guide.md) | Promtail 설정과 pipeline stages |
| 6 | [integrations/grafana-integration-guide.md](integrations/grafana-integration-guide.md) | Grafana 데이터소스, Explore, 대시보드 |
| 7 | [operations/alerting-guide.md](operations/alerting-guide.md) | Loki ruler와 알림 규칙 |
| 8 | [operations/retention-guide.md](operations/retention-guide.md) | 스토리지와 보존 정책 |
| 9 | [tutorials/log-query-test.md](tutorials/log-query-test.md) | 로그 조회와 알림 실습 |

## 전체 문서 목록

| 구분 | 문서 |
|------|------|
| 설치 | [install/install.md](install/install.md), [install/install-helm.md](install/install-helm.md), [install/install-systemd.md](install/install-systemd.md), [install/install-docker-compose.md](install/install-docker-compose.md), [install/upgrade/README.md](install/upgrade/README.md) |
| 핵심 개념 | [concepts/label-guide.md](concepts/label-guide.md) |
| LogQL | [logql/logql-basics-guide.md](logql/logql-basics-guide.md), [logql/logql-advanced-guide.md](logql/logql-advanced-guide.md) |
| 수집 | [collection/promtail-guide.md](collection/promtail-guide.md) |
| 연동 | [integrations/grafana-integration-guide.md](integrations/grafana-integration-guide.md) |
| 운영 | [operations/alerting-guide.md](operations/alerting-guide.md), [operations/retention-guide.md](operations/retention-guide.md) |
| 실습 | [tutorials/log-query-test.md](tutorials/log-query-test.md) |
| 문서 운영 | [rules/README.md](rules/README.md), [templates/README.md](templates/README.md), [agents/README.md](agents/README.md) |
| 실행 자산 | [../ops/README.md](../ops/README.md) |

## 폴더 역할

| 폴더 | 역할 |
|------|------|
| [install/](install/README.md) | 설치 방식별 절차와 업그레이드 |
| [concepts/](concepts/README.md) | 레이블 설계와 카디널리티 관리 |
| [logql/](logql/README.md) | LogQL 기초와 심화 쿼리 |
| [collection/](collection/README.md) | Promtail 로그 수집 파이프라인 |
| [integrations/](integrations/README.md) | Grafana 연동 |
| [operations/](operations/README.md) | 알림, 보존 정책, 스토리지 운영 |
| [tutorials/](tutorials/README.md) | 단계별 실습 |
| [rules/](rules/README.md) | 문서 작성, Loki 컨벤션, 보안, 모니터링 규칙 |
| [templates/](templates/README.md) | 서비스 문서, 런북, 장애 보고서 템플릿 |
| [agents/](agents/README.md) | AI가 문서를 작성하거나 진단할 때 참고할 역할별 지침 |

## 관리 원칙

- 설치 문서는 `install/`, 핵심 개념은 `concepts/`, LogQL 문서는 `logql/`, 수집 문서는 `collection/`에 둡니다.
- Grafana 같은 외부 도구 연동은 `integrations/`, 운영 정책은 `operations/`, 직접 따라 하는 실습은 `tutorials/`에 둡니다.
- 실제 적용 가능한 Helm values, 알림 규칙, 샘플 앱 매니페스트는 `ops/`에 둡니다.
- 새 문서를 추가할 때는 이 README의 전체 문서 목록과 추천 읽기 순서를 함께 갱신합니다.
