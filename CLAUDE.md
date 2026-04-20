# loki-practice — 프로젝트 가이드

## 프로젝트 설정
- 환경: EKS
- Loki 버전: 3.x (Helm chart: grafana/loki)
- Promtail 버전: 3.x (Helm chart: grafana/promtail)
- 네임스페이스: monitoring
- 앱 이름 컨벤션: my-app

---

## 디렉토리 구조

```
loki-practice/
├── CLAUDE.md                  # 이 파일 (자동 로드)
├── .claude/
│   ├── settings.json
│   └── commands/              # /new-doc, /new-runbook, /review-doc, /add-troubleshooting, /search-kb
├── agents/                    # doc-writer, label-designer, logql-advisor, troubleshooter
├── templates/                 # service-doc, runbook, incident-report
├── rules/                     # doc-writing, loki-conventions, security-checklist, monitoring
├── loki/                      # Loki 설정 파일
├── app/                       # 테스트 애플리케이션
└── *-guide.md                 # 주제별 가이드 문서
```

---

## 커스텀 슬래시 명령어

| 명령어 | 설명 | 사용 예시 |
|--------|------|---------|
| `/new-doc` | 새 가이드 문서 생성 | `/new-doc structured-metadata` |
| `/new-runbook` | 새 런북 생성 | `/new-runbook Loki 스토리지 포화 대응` |
| `/review-doc` | 문서 검토 | `/review-doc label-guide.md` |
| `/add-troubleshooting` | 트러블슈팅 케이스 추가 | `/add-troubleshooting 로그 수집 누락` |
| `/search-kb` | 지식베이스 검색 | `/search-kb LogQL 메트릭 쿼리` |

---

## 가이드 문서 목록

| 문서 | 주제 |
|------|------|
| `install.md` | Loki 설치 (Helm + EKS) |
| `label-guide.md` | 레이블 설계 및 카디널리티 |
| `logql-basics-guide.md` | LogQL 기초 |
| `logql-advanced-guide.md` | LogQL 고급 (메트릭 쿼리) |
| `promtail-guide.md` | Promtail 수집 설정 |
| `grafana-integration-guide.md` | Grafana 연동 |
| `alerting-guide.md` | Loki 알림 규칙 |
| `retention-guide.md` | 보존 정책 |
| `log-query-test.md` | 로그 쿼리 테스트 실습 |

---

## 핵심 명령어

```bash
# Loki 상태 확인
kubectl get pods -n monitoring -l app=loki

# logcli로 로그 조회
logcli query '{namespace="default"}' --limit=20

# Loki 레이블 목록
logcli labels

# Loki 헬스 체크
curl http://loki:3100/ready
```
