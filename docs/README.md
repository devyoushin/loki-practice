# Loki Docs

Loki를 처음 보는 사람이 설치부터 로그 수집과 쿼리까지 따라갈 수 있도록 문서를 묶어 둔 디렉터리다.

## 어디서 시작할까

| 순서 | 문서 | 용도 |
|------|------|------|
| 1 | `install.md` | Helm, systemd, Docker Compose 설치 방식 |
| 2 | `label-guide.md` | 레이블 설계와 카디널리티 관리 |
| 3 | `logql-basics-guide.md`, `logql-advanced-guide.md` | LogQL 조회와 집계 |
| 4 | `promtail-guide.md` | 수집 파이프라인과 pipeline stages |
| 5 | `grafana-integration-guide.md` | Grafana 연결과 탐색 |
| 6 | `alerting-guide.md`, `retention-guide.md` | 알림과 보존 정책 |
| 7 | `log-query-test.md` | 쿼리 실습과 검증 |
| 8 | `rules/README.md` | 문서와 운영 규칙 |
| 9 | `agents/README.md` | AI 작업 지침 |
| 10 | `templates/README.md` | 문서 템플릿 |
| 11 | `../ops/README.md` | 실제 실행 자산과 운영 방법 |

## 문서 구조

| 구분 | 문서 |
|------|------|
| 설치/기초 | `install.md`, `label-guide.md` |
| 쿼리 | `logql-basics-guide.md`, `logql-advanced-guide.md`, `log-query-test.md` |
| 수집/연동 | `promtail-guide.md`, `grafana-integration-guide.md` |
| 운영 | `alerting-guide.md`, `retention-guide.md` |
| 보조 자료 | `rules/`, `agents/`, `templates/` |

## 읽는 순서

1. `install.md`
2. `label-guide.md`
3. `logql-basics-guide.md`
4. `logql-advanced-guide.md`
5. `promtail-guide.md`
6. `grafana-integration-guide.md`
7. `alerting-guide.md`
8. `retention-guide.md`
9. `rules/README.md`
10. `agents/README.md`
11. `templates/README.md`

## 관련 경로

- `rules/`는 문서/운영 규칙
- `agents/`는 Claude 작업 지침
- `templates/`는 반복 문서 골격
- `../ops/`는 Loki/Promtail values와 샘플 앱
