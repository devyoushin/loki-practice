# Loki Docker Compose 설치

로컬 개발이나 빠른 쿼리 테스트용으로 Loki를 `docker compose`로 띄울 때 사용한다.

## 대상

- `loki`
- `promtail`
- `grafana`

## 절차

1. `compose.yaml`에 이미지, 포트, volume을 정의한다.
2. Loki 설정과 Promtail pipeline 설정을 bind mount 한다.
3. `docker compose up -d`로 올린다.
4. `docker compose logs -f`와 `docker compose ps`로 확인한다.

## 확인 명령

```bash
docker compose ps
curl http://localhost:3100/ready
curl http://localhost:3000/api/health
```

## 운영 포인트

- `promtail`은 file target과 positions 파일을 반드시 둔다.
- 운영용 보존 정책은 Helm 문서를 따른다.
