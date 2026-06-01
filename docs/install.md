# Loki 설치 안내

설치 방식별 문서를 분리해 둔 입구다.

## 설치 방식

| 방식 | 문서 | 설명 |
|------|------|------|
| Helm | `install-helm.md` | Loki, Promtail, Grafana를 EKS에 설치 |
| systemd | `install-systemd.md` | 단일 VM/베어메탈에서 Loki와 Promtail을 서비스로 관리 |
| Docker Compose | `install-docker-compose.md` | 로컬 로그 스택 검증용 |

## 읽는 순서

1. `install-helm.md`
2. `install-systemd.md`
3. `install-docker-compose.md`
