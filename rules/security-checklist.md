# 보안 체크리스트 — loki-practice

## Loki 보안 설정

### 인증/인가
- [ ] Loki API 인증 설정 (Basic Auth 또는 Grafana 프록시)
- [ ] 멀티 테넌시 활성화 (`auth_enabled: true`)
- [ ] 테넌트별 접근 격리 확인
- [ ] `X-Scope-OrgID` 헤더 검증

### 데이터 보안
- [ ] 민감 데이터 마스킹 (Promtail pipeline_stages replace)
  - 비밀번호, 토큰, 신용카드 번호
  - 주민등록번호, 전화번호
- [ ] 로그에 포함된 자격증명 정기 감사
- [ ] 스토리지(S3) 버킷 암호화 활성화
- [ ] 스토리지 버킷 퍼블릭 접근 차단

### 네트워크 보안
- [ ] Loki API 포트(3100) 외부 노출 금지
- [ ] Promtail → Loki TLS 통신
- [ ] NetworkPolicy로 접근 제한

### 스토리지 보안
- [ ] S3 IRSA 또는 IAM Role 인증 (장기 자격증명 금지)
- [ ] S3 버킷 정책으로 Loki 서비스계정만 접근 허용
- [ ] 청크 암호화 설정 확인

## 정기 보안 점검 (월별)
- [ ] 로그 내 민감 데이터 노출 여부 감사
- [ ] 스토리지 접근 로그 검토
- [ ] Loki 버전 보안 업데이트 확인
- [ ] 테넌트 접근 권한 리뷰
