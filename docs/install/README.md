# 설치 문서

Loki 설치 방식별 절차와 업그레이드 문서를 모은 폴더입니다.

## 문서 목록

| 문서 | 용도 |
|------|------|
| [install.md](install.md) | Helm, systemd, Docker Compose 설치 방식 선택 |
| [install-helm.md](install-helm.md) | EKS 기준 Loki, Promtail, Grafana Helm 설치 |
| [install-systemd.md](install-systemd.md) | VM 또는 단일 서버 systemd 설치 |
| [install-docker-compose.md](install-docker-compose.md) | 로컬 Docker Compose 검증 |
| [upgrade/README.md](upgrade/README.md) | Loki 업그레이드와 롤백 |

## 관련 경로

- [../../ops/config/loki/](../../ops/config/loki/)에는 Loki/Promtail Helm values와 알림 규칙이 있습니다.
- 설치 후 로그 수집 설정은 [../collection/promtail-guide.md](../collection/promtail-guide.md)에서 이어갑니다.
