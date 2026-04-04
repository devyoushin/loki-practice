# loki-practice

A hands-on repository for learning Loki on EKS.
- **Environment**: EKS / Loki 3.x
- **Namespaces**: log stack `monitoring`, app `default`

---

## Learning Path

```
1. Installation    → install.md
2. Core Concepts   → logql-basics-guide.md, label-guide.md
3. Advanced
   ├── Parsing     → logql-advanced-guide.md
   ├── Collection  → promtail-guide.md
   ├── Alerting    → alerting-guide.md
   └── Storage     → retention-guide.md
4. Visualization   → grafana-integration-guide.md
5. Hands-on        → log-query-test.md
```

---

## Documents

### Installation
| File | Description |
|------|-------------|
| [install.md](./install.md) | Install Loki + Promtail + Grafana via Helm |

### Core Concepts
| File | Description |
|------|-------------|
| [logql-basics-guide.md](./logql-basics-guide.md) | LogQL 기초 — 로그 스트림 선택, 라인 필터, 레이블 필터 |
| [label-guide.md](./label-guide.md) | 레이블 설계 — 카디널리티, 정적/동적 레이블 |

### Advanced
| File | Description |
|------|-------------|
| [logql-advanced-guide.md](./logql-advanced-guide.md) | LogQL 심화 — JSON/logfmt 파싱, 메트릭 쿼리, 집계 |
| [promtail-guide.md](./promtail-guide.md) | Promtail — Pipeline stages, 레이블 추출, 멀티라인 처리 |
| [alerting-guide.md](./alerting-guide.md) | Loki Alerting — PrometheusRule로 로그 기반 알림 설정 |
| [retention-guide.md](./retention-guide.md) | 보존 정책 — 스토리지 설정, 보존 기간, compactor |

### Visualization
| File | Description |
|------|-------------|
| [grafana-integration-guide.md](./grafana-integration-guide.md) | Grafana 연동 — Loki 데이터소스, Explore, 대시보드 |

### Hands-on
| File | Description |
|------|-------------|
| [log-query-test.md](./log-query-test.md) | 단계별 LogQL 실습 (기본 조회 → 파싱 → 메트릭 → 알림) |

---

## Manifest Structure

```
app/
├── deployment.yaml    # 로그를 생성하는 샘플 앱 (info/warn/error/JSON)
└── service.yaml       # 샘플 앱 Service

loki/
├── loki-values.yaml       # Loki Helm values (SingleBinary 모드)
├── promtail-values.yaml   # Promtail Helm values
└── alerting-rule.yaml     # 로그 기반 알림 규칙 예시
```

---

## Key Concept Summary

**Promtail → Loki → Grafana** 가 Loki 스택의 핵심 흐름입니다.

```
[App Pod]
   │ stdout/stderr
   ▼
[Promtail]       → 로그 수집, 레이블 부착, Loki로 전송
   │ push (HTTP)
   ▼
[Loki]           → 레이블 인덱싱 + 로그 청크 저장
   │
   ▼
[Grafana]        → LogQL로 조회, 대시보드, 알림
```

> Loki는 로그 **내용을 인덱싱하지 않고** 레이블만 인덱싱합니다.
> 덕분에 Elasticsearch보다 훨씬 저렴하게 운영할 수 있습니다.
