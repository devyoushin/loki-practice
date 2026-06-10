# Loki Ops

Loki를 실제로 설치하거나 실습할 때 사용하는 Helm values, 알림 규칙, 샘플 애플리케이션 매니페스트를 두는 공간입니다. 개념 설명은 `docs/`에, 적용 가능한 실행 자산은 `ops/`에 둡니다.

## 폴더 구조

| 폴더 | 내용 |
|------|------|
| `config/loki/` | Loki/Promtail Helm values, 로그 기반 알림 규칙 |
| `config/app/` | 로그를 생성하는 샘플 애플리케이션 매니페스트 |

## 관련 문서

| 작업 | 문서 |
|------|------|
| 설치 방식 선택 | [../docs/install/install.md](../docs/install/install.md) |
| Helm 설치 | [../docs/install/install-helm.md](../docs/install/install-helm.md) |
| 레이블 설계 | [../docs/concepts/label-guide.md](../docs/concepts/label-guide.md) |
| Promtail 수집 설정 | [../docs/collection/promtail-guide.md](../docs/collection/promtail-guide.md) |
| Grafana 연동 | [../docs/integrations/grafana-integration-guide.md](../docs/integrations/grafana-integration-guide.md) |
| 알림 규칙 | [../docs/operations/alerting-guide.md](../docs/operations/alerting-guide.md) |
| 로그 쿼리 실습 | [../docs/tutorials/log-query-test.md](../docs/tutorials/log-query-test.md) |

## 관리 원칙

- 재사용 가능한 Helm values, 알림 규칙, 샘플 앱 매니페스트는 이 디렉터리에 둡니다.
- 문서 본문에는 핵심 스니펫만 넣고, 전체 적용 파일은 `ops/config/`를 기준으로 관리합니다.
- 설정을 바꾸면 관련 문서의 명령어와 파일 경로도 함께 확인합니다.
