Loki 트러블슈팅 케이스를 추가합니다.

**사용법**: `/add-troubleshooting <증상 설명>`  **예시**: `/add-troubleshooting 로그 수집 지연`

형식:
```markdown
### <증상>
**원인**: <근본 원인>
**확인**:
\`\`\`bash
kubectl logs -n monitoring -l app.kubernetes.io/name=loki
logcli query '{app="my-app"}' --limit=10
\`\`\`
**해결**: <해결 방법>
```
