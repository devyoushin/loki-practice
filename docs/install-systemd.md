# Loki systemd 설치

단일 VM이나 베어메탈에서 Loki와 Promtail을 systemd 서비스로 관리할 때 사용한다.

## 대상

- `loki.service`
- `promtail.service`

## 준비물

- Loki/Promtail 바이너리
- 설정 파일: `/etc/loki/config.yml`, `/etc/promtail/config.yml`
- 데이터/로그 디렉터리: `/var/lib/loki`, `/var/lib/promtail`

## 절차

1. 바이너리를 설치한다.
2. 설정 파일과 positions 파일 경로를 만든다.
3. unit 파일을 `/etc/systemd/system/`에 둔다.
4. `systemctl enable --now`로 시작한다.
5. `journalctl -u loki -f`, `journalctl -u promtail -f`로 로그를 본다.

## 확인 명령

```bash
systemctl status loki
systemctl status promtail
curl http://localhost:3100/ready
```

## 운영 포인트

- retention과 compactor 설정을 함께 확인한다.
- Promtail은 노드별로 positions 파일을 유지해야 한다.
- Grafana 연동은 별도 문서에서 관리한다.
