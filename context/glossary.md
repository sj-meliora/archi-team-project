# 용어집 / ID 매핑 (Glossary)

> category: meta | updated: 2026-06-19

## 약어
| 약어 | 의미 |
|---|---|
| NPU | Neural Processing Unit (On-device AI 가속기) |
| IR | Intermediate Representation (중간 표현) |
| SDK | On-device AI SDK = IR Converter → Graph Optimizer → Quantizer → Compiler |
| HITL | Human-In-The-Loop (사람 개입 승인) |
| SPOF | Single Point Of Failure (단일 장애점) |
| MTTR | Mean Time To Recovery (평균 복구 시간) |
| E2E | End-to-End |
| NFS | Network File System |

## ⚠️ QA 번호 슬라이드별 매핑 (정합성 #1)
canonical = **팀 합의 우선순위 기준** (2026-06-23 재정렬, QA·QAS 2자리). 슬라이드마다 같은 번호가 다른 속성을 가리키므로 주의.

| canonical (현재 우선순위) | 슬라이드 11(Driver) | 슬라이드 25~28(QA상세) |
|---|---|---|
| QA-01 Scalability | QA02 확장성 | QA02 확장성 |
| QA-02 Availability(운영안정) | QA03 **Agent 수행시간** | QA03 **Agent 수행시간** |
| QA-03 Controllability | QA04 안정적 운영 | QA04 운영 안정성 |
| QA-05 Efficiency(토큰) | QA01 토큰 | QA01 토큰 |
| QA-07 Performance(Agent수행시간) | — | — |

→ 발표 자료는 canonical(QA-01~10) 한 체계로 통일하고, 위 매핑은 원본 대조용으로만 보존.
