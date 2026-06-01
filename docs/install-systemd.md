# Loki systemd 설치

단일 VM이나 베어메탈에서 Loki와 Promtail을 systemd 서비스로 관리할 때 사용한다.

## 대상

- `loki.service`
- `promtail.service`

## 준비물

- Loki/Promtail 바이너리
- 설정 파일: `/etc/loki/config.yml`, `/etc/promtail/config.yml`
- 데이터/로그 디렉터리: `/var/lib/loki`, `/var/lib/promtail`

## RPM 설치

Loki와 Promtail은 환경에 따라 공식 패키지보다 내부 RPM으로 배포하는 경우가 많다. 내부 저장소나 사내 빌드 산출물이 있다면 버전을 고정해서 설치한다.

```bash
sudo dnf install -y ./loki-<VERSION>-1.x86_64.rpm
sudo dnf install -y ./promtail-<VERSION>-1.x86_64.rpm
rpm -qa | grep -E 'loki|promtail'
```

패키지가 unit 파일을 포함하는지 확인한다.

```bash
rpm -ql loki | grep systemd
systemctl cat loki
```

## tarball 설치

공식 릴리스 바이너리를 내려받아 `/usr/local/bin`에 배치하는 방식이다.

```bash
LOKI_VERSION="<VERSION>"
curl -LO "https://github.com/grafana/loki/releases/download/v${LOKI_VERSION}/loki-linux-amd64.zip"
curl -LO "https://github.com/grafana/loki/releases/download/v${LOKI_VERSION}/promtail-linux-amd64.zip"
unzip loki-linux-amd64.zip
unzip promtail-linux-amd64.zip
sudo install -m 0755 loki-linux-amd64 /usr/local/bin/loki
sudo install -m 0755 promtail-linux-amd64 /usr/local/bin/promtail
sudo mkdir -p /etc/loki /etc/promtail /var/lib/loki /var/lib/promtail
```

## 절차

1. RPM 또는 tarball 방식 중 하나로 바이너리를 설치한다.
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
