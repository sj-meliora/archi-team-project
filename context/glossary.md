# 용어집 / ID 매핑 (Glossary)

> category: meta | updated: 2026-06-24

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

> **2026-06-24 재번호(OI-8)**: 신규 QA 3종 정식 편입 — NQA-A→**QA-06 Security**, NQA-B→**QA-07 Correctness**(6·7위 삽입), NQA-C→**QA-13 Cost**. 기존 QA-06~10은 +2 시프트(→QA-08~12). 아래 표는 원본 슬라이드 대조용(구 매핑)이라 슬라이드-origin이 없는 QA-06/07/13은 행이 없다. 현재 전체 번호는 `INDEX.md`·각 `QA-*.md`가 SSoT.

| canonical (현재 우선순위) | 슬라이드 11(Driver) | 슬라이드 25~28(QA상세) |
|---|---|---|
| QA-01 Scalability | QA02 확장성 | QA02 확장성 |
| QA-02 Availability(운영안정) | QA03 **Agent 수행시간** | QA03 **Agent 수행시간** |
| QA-03 Controllability | QA04 안정적 운영 | QA04 운영 안정성 |
| QA-05 Efficiency(토큰) | QA01 토큰 | QA01 토큰 |
| QA-09 Performance(Agent수행시간) | — | — |

→ 발표 자료는 canonical(QA-01~13) 한 체계로 통일하고, 위 매핑은 원본 대조용으로만 보존.
