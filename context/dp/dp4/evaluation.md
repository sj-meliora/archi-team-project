# DP-0004 통합 평가 — 7개 대안 비교 및 조합 권고

> category: DP-eval | for: DP-0004 | updated: 2026-06-20
> 기존 1·2안 + 신규 A3~A7(패턴 기반 발굴)을 driving QA 기준으로 통합 비교한다.
> driving QA: QA-0002(Scalability), QA-0006(Reliability-Workflow), QA-0008(Performance-E2E), QA-0010(Maintainability).

## 핵심 통찰 — DP-0004는 "단일 택1"이 아니라 4개의 직교 결정 축
신규 대안을 발굴하며, 원래 1안 vs 2안으로 보였던 결정이 사실 **서로 직교하는 4개 축**임을 식별했다. 중간 피드백("architecture pattern 보강")의 핵심 가치도 여기에 있다 — 단일 비교가 아니라 **축별로 패턴을 조합**하는 설계.

| 결정 축 | 질문 | 관련 대안 |
|---|---|---|
| **A. 실행 위치/자원 모델** | 노드를 어디서 실행? | 1안(타입 풀) · 2안(노드 인스턴스) · **A5**(ephemeral) |
| **B. 작업 분배 메커니즘** | 노드 간 작업을 어떻게 전달? | (동기 라우팅) · **A3**(비동기 큐/Choreography) · **A6**(Orchestration) |
| **C. 구조적 분해** | 파이프라인을 어떻게 쪼갬? | **A4**(무상태 filter) |
| **D. 변종/복구 관심사** | 구현 다양성·실행 내구성? | **A7**(변종 plugin) · **A6**(내구 실행) |

## 통합 Trade-off 매트릭스
| # | 대안 | 근거 패턴 | Perf(E2E) | Scalability | Reliability-WF | Maintainability |
|---|---|---|:---:|:---:|:---:|:---:|
| 1안 | 타입별 공유 서버 풀 | Shared resource pool | ★★☆ | ★★★ | ★★☆ | ★☆☆ |
| 2안 | 노드당 Workflow 인스턴스 | Bulkhead | ★★★ | ★★☆ | ★★★ | ★★★ |
| **A3** | Event-Driven Work Queue | Competing Consumers, Claim-Check | ★★☆ | ★★★ | ★★☆ | ★★★ |
| **A4** | Pipes-and-Filters Stage | Pipes and Filters, Stateless | ★★☆ | ★★★ | ★★☆ | ★★★ |
| **A5** | Serverless/Ephemeral Node | FaaS, Scale-to-zero, Bulkhead | ★★☆ | ★★★ | ★★★ | ★★☆ |
| **A6** | Durable Orchestration | Event Sourcing, Saga | ★★☆ | ★★☆ | ★★★ | ★★★ |
| **A7** | Microkernel Plug-in Node | Microkernel, Plugin registry | ★★☆ | ★★★ | ★★☆ | ★★★ |

## QA별 최강 대안
- **Scalability(QA-0002)**: A5(인스턴스 탄력) + A7(변종 다양성) — "수량"과 "종류" 확장을 분담.
- **Reliability-WF(QA-0006)**: A6(내구 복구) ≥ A5(invocation 격리) ≥ 2안(노드 격리).
- **Performance-E2E(QA-0008)**: 2안(네트워크 전달 없음)이 우위, 신규 대안은 hop/cold-start로 ★★☆ — claim-check·pre-warm로 보완 필요.
- **Maintainability(QA-0010)**: A4(분해) · A7(변종 교체) · A6(흐름 명시) 동급 우위. 1안이 최약.

## 조합 권고 (단일 택1이 아닌 패턴 스택)
중간 피드백을 반영해, **단일 안 선택 대신 축별 조합**을 권고안으로 제시한다.

- **권고 스택 = A4(구조) + A3(분배) + A5(실행) + A7(변종)**, 옵션으로 A6(내구):
  - **A4 Pipes-and-Filters**로 4단계를 무상태 filter로 분해(Maintainability 기반).
  - **A3 메시지 큐**를 pipe 구현체로 사용 → A4의 monolithic 파이프라인 실패전파 위험을 큐 버퍼+DLQ로 상쇄(Reliability 보완).
  - **A5 ephemeral 실행**으로 각 filter를 scale-to-zero 호출 → 2안의 자원낭비(R-2) 해소(Scalability·활용률).
  - **A7 plugin**으로 NPU 타겟·모델 변종을 수용(variety 확장).
  - (옵션) **A6 워크플로 엔진**을 상위에 두면 crash 복구·Saga 보상으로 Reliability 최상위 — 단 DP-0002 오케스트레이터와 정합 필요.
- **이 스택의 효과**: 2안의 강점(격리·Maintainability)을 유지하면서 1안의 약점(Maintainability ★☆☆)과 2안의 약점(Scalability 활용률)을 동시에 해소.

## 미해결/검증 필요 (open-issues 연계)
- **전달 오버헤드 ≤5%(QA-0008)**: A3/A5/A6 모두 hop·cold-start 추가 → claim-check + pre-warm 효과 정량 검증 필요.
- **공유 의존점 SPOF**: A3 브로커 / A6 오케스트레이터 → 다중화·샤딩 전제. DP-0002와 정합.
- **plugin API 안정성(A7)**: 초기 계약 설계가 critical(breaking change 리스크).
- **DP-0001·DP-0005 정합**: A5↔DP-0001(Dynamic Pool), A3/A6↔DP-0005(공유 캐시·memoization) 일관 결정 필요.

## 출처 (리서치 근거)
- R-01 [Azure Competing Consumers](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers) · [Claim-Check](https://learn.microsoft.com/en-us/azure/architecture/patterns/claim-check)
- R-02 [Azure Pipes and Filters](https://learn.microsoft.com/en-us/azure/architecture/patterns/pipes-and-filters) · [Knative/Serverless](https://www.cloudraft.io/blog/building-serverless-functions-on-kubernetes-using-knative)
- R-03 [Temporal Durable Execution](https://temporal.io/blog/temporal-replaces-state-machines-for-distributed-applications) · [Saga Orchestration vs Choreography](https://temporal.io/blog/to-choreograph-or-orchestrate-your-saga-that-is-the-question)
- R-04 [Microkernel Architecture](https://softwaresystemdesign.com/software-architecture-design/architectural-patterns/microkernel-architecture/)
