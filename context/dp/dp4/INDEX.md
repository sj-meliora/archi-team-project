# DP-0004 Approach 보강 작업 색인

> category: DP-meta | owner: 17조(담당: 본인) | updated: 2026-06-20
> 목적: 중간 피드백("architecture pattern 보강된 approach가 더 나왔으면") 반영.
> 검증된 아키텍처 패턴 카탈로그에 근거하여 DP-0004의 **추가 대안(3안~)** 을 발굴·평가한다.

## DP-0004 재확인 (대상 결정)
- **결정 포인트**: 파이프라인 노드 작업(IR Converter → Graph Optimizer → Quantizer → Compiler)을 **어떤 실행 구조**로 처리할 것인가.
- **driving QA**: QA-0002(Scalability), QA-0006(Reliability-Workflow), QA-0008(Performance-E2E), QA-0010(Maintainability).
- **기존 대안**: 1안 타입별 공유 서버 풀 / 2안 노드당 Workflow 인스턴스. → 본 작업은 여기에 **패턴 기반 신규 대안**을 추가한다.
- **인접 정합 제약**: DP-0001(Agent 배치), DP-0005(캐시 전략)와 격리·확장 정책이 충돌하지 않아야 함.

## 폴더 구조
```
dp4/
├── INDEX.md          (이 파일) 색인·iteration 로그·approach 상태표
├── research/         아키텍처 패턴 리서치 문서 (R-NN)
├── approaches/       발굴한 추가 대안 (A-N안, DP-0004 후보 대안)
└── evaluation.md     전 대안 통합 trade-off / ATAM 비교 (1안·2안 + 신규)
```

## 발굴 Approach 상태표
| ID | 이름 | 근거 패턴 | 리서치 | 상태 |
|---|---|---|---|---|
| (1안) | 타입별 공유 서버 풀 | Shared resource pool | — | 기존(DP-0004 본문) |
| (2안) | 노드당 Workflow 인스턴스 | Bulkhead | — | 기존(DP-0004 본문) |
| A3(3안) | Event-Driven Work Queue | Competing Consumers, Claim-Check, EDA | R-01 | 발굴·평가완료 |
| A4(4안) | Pipes-and-Filters Stateless Stage | Pipes and Filters, Stateless, Std schema | R-02 | 발굴·평가완료 |
| A5(5안) | Serverless/Ephemeral Compute-per-Node | FaaS, Ephemeral, Scale-to-zero, Bulkhead | R-02 | 발굴·평가완료 |

## Iteration 로그
- **Iter1 (2026-06-20)**: Event-driven/message-queue 패턴군 리서치(R-01) → A3 Event-Driven Work Queue 발굴·평가. 근거: Azure Competing Consumers/Claim-Check.
- **Iter2 (2026-06-20)**: 구조 스타일(Pipes-and-Filters) + 실행 기반(Serverless/Ephemeral) 리서치(R-02) → A4·A5 발굴·평가. 결정 축이 "어디서 실행(A5)"과 "어떻게 분해(A4)" 둘로 분리됨을 식별. 근거: Azure Pipes-and-Filters, Knative/FaaS.
