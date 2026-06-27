# DP-0004 통합 평가 — 7개 대안 비교 및 조합 권고

> category: DP-eval | for: DP-0004 | updated: 2026-06-20
> 기존 1·2안 + 신규 A3~A7(패턴 기반 발굴)을 driving QA 기준으로 통합 비교한다.
> driving QA: QA-01(Scalability), QA-08(Reliability-Workflow), QA-10(Performance-E2E), QA-12(Maintainability).

## 핵심 통찰 — DP-0004는 "단일 택1"이 아니라 4개의 직교 결정 축
신규 대안을 발굴하며, 원래 1안 vs 2안으로 보였던 결정이 사실 **서로 직교하는 4개 축**임을 식별했다. 중간 피드백("architecture pattern 보강")의 핵심 가치도 여기에 있다 — 단일 비교가 아니라 **축별로 패턴을 조합**하는 설계.

| 결정 축 | 질문 | 관련 대안 |
|---|---|---|
| **A. 실행 위치/자원 모델** | 노드를 어디서 실행? | 1안(타입 풀) · 2안(노드 인스턴스) · **A5**(ephemeral 원격) · **A8**(ephemeral 로컬) |
| **B. 작업 분배 메커니즘** | 노드 간 작업을 어떻게 전달? | (동기 라우팅) · **A3**(비동기 큐/Choreography) · **A6**(Orchestration) |
| **C. 구조적 분해** | 파이프라인을 어떻게 쪼갬? | **A4**(무상태 filter) |
| **D. 변종/복구 관심사** | 구현 다양성·실행 내구성? | **A7**(변종 plugin) · **A6**(내구 실행) |

> **축 A 세분화(`decision-axes.md`)**: 리뷰 수렴 작업에서 축 A를 다시 **2개 직교 손잡이**(수명: 상주↔일회용 / 배치: 원격↔로컬)로 분해해 1안·2안·A5·A8을 2x2에 고정했다. 결론: 각 계열 진화형인 **A5(일회용·원격) vs A8(일회용·로컬)** 가 최종 대결이고, 1안·2안은 지배당하는 기준선으로 격하. 상세 분류·스펙트럼 통찰은 `decision-axes.md` 참조.

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
| **A8** | Co-located Ephemeral Stage | Data-locality, Co-location, Pipeline parallelism | ★★★ | ★★★ | ★★☆ | ★★☆ |

> A8 별점 근거: 로컬 전달로 Performance ★★★(2안 강점 계승) + 단계 입도 확장으로 Scalability ★★★. 단 로컬 디스크 내구성 약점으로 Reliability ★★☆, 배치 제어 복잡성으로 Maintainability ★★☆. (상세 `approaches/A8-*`)

## QA별 최강 대안
- **Scalability(QA-01)**: A5·A8(둘 다 일회용 탄력) — A5는 클러스터 폭증, A8은 노드 내 단계 입도. A7(변종)은 별도 축(variety, 수량 아님).
- **Reliability-WF(QA-08)**: A6(내구 복구) ≥ A5(invocation 격리·스토리지 보존) ≥ 2안(노드 격리) > A8(단계 격리이나 로컬 디스크 유실 위험).
- **Performance-E2E(QA-10)**: 2안·**A8(로컬 전달)** 이 우위(★★★), 원격 계열(A3/A5/A6)은 claim-check 전달세금으로 ★★☆ — pre-warm·co-location로 보완 필요.
- **Maintainability(QA-12)**: A4(분해) · A7(변종 교체) · A6(흐름 명시) 동급 우위. 1안이 최약.

## 수렴 결론 (리뷰 F-10 반영 — "다 쌓기"에서 택일로)
> ⚠️ 종전 "권고 스택(A4+A3+A5+A7+A6 다 쌓기)"은 리뷰(F-02/F-03/F-10)에서 **과설계·이중 제어평면·복합 trade-off 미검증**으로 결론 부적합 판정. 아래 수렴안으로 대체한다. ATAM의 본령은 **버릴 것을 정하는 것**.

### 1) 핵심 결정 — 축 A에서 A5 vs A8로 수렴
- `decision-axes.md`의 2x2로 축 A를 재정렬하면 **1안→A5, 2안→A8** 로 각 계열이 진화형에 지배된다 → **1안·2안은 기준선(null)으로 격하**, 후보는 **A5(일회용·원격) vs A8(일회용·로컬)** 둘로 압축.
- **가르는 단일 질문**: `(4단계 × 20GB 왕복) ÷ 스토리지 대역폭`이 E2E의 5% budget(QA-10) 안에 드는가.
  - 든다 → **A5**(단순·복구 유리, 전달세금 감내).
  - 넘는다 → **A8**(로컬 전달로 세금 회피, locality-first).
- 지배 드라이버 **Scalability(H)+Reliability(M)** 기준 둘 다 강하나, **Performance(전달) ↔ Reliability(복구)** 교환에서 갈림(A8=전달 우위/복구 약, A5=복구 우위/전달 약).

### 2) 직교 보강축(A4·A6·A7)은 "조건부로 얹기" — 동시 채택 아님
- **A4(구조 분해)**: 선택된 실행기반 위 **설계 규율**로만 채택(신규 인프라 아님). A8과는 로컬 스트리밍 모드로 결합.
- **A3 vs A6(제어평면)**: **동시 채택 금지**(이중 제어평면 안티패턴, F-03). 이 축은 **DP-01(Agent Hierarchy)와 동시 결정** — Orchestration이면 A6, Choreography면 A3로 단일화.
- **A7(변종)**: Scalability 아님(variety=Extensibility). **변종 수가 임계치 초과 시에만** 도입.

### 3) 단계적 채택 로드맵 + 배제 기준 (발표 결론 슬라이드 골격)
```
0) 기준선: 2안(로컬 격리) — 비교 기준으로 유지
1) 실행기반 택일: 5% 산식 → A5(원격) 또는 A8(로컬)
2) A4 분해 규율 적용 (단계 입도 확장의 전제)
   [트리거] 제어 가시성/복구가 binding → DP-01와 함께 A3 or A6 택1
   [트리거] 변종 수 > N → A7
```
- **배제 기준**: cold-start를 X ms 이하로 못 누르면 A5 보류 / 노드가 모델 파이프라인을 못 담으면 A8 보류 / 변종 < N이면 A7 미도입.

## 미해결/검증 필요 (open-issues 연계)
- **🔑 5% 산식이 A5 vs A8을 가름(QA-10)**: `(4단계 × 20GB 왕복) ÷ 스토리지 대역폭` 실측·노드 사양 의존 → **팀 검증 대상**(이 수치가 최종 택일을 결정).
- **A8 로컬 디스크 내구성(QA-08)**: 노드 사망 시 산출물 유실 → 복제/체크포인트 정책 필요 여부 판정.
- **A8 노드 footprint**: 모델당 파이프라인이 노드에 들어가는지(안 들어가면 spill → A5 수렴) 검증.
- **잔여 P0(리뷰)**: 가중 매트릭스(H/M/M/L)·간이 FMEA·멱등성↔QA-11 모순 해소는 **미반영** — 다음 수렴 iteration 과제.
- **전달 오버헤드 ≤5%(QA-10)**: A3/A5/A6 모두 hop·cold-start 추가 → claim-check + pre-warm 효과 정량 검증 필요.
- **공유 의존점 SPOF**: A3 브로커 / A6 오케스트레이터 → 다중화·샤딩 전제. DP-01와 정합.
- **plugin API 안정성(A7)**: 초기 계약 설계가 critical(breaking change 리스크).
- **구 DP-0001·DP-0005 정합**: A5↔구 DP-0001(Dynamic Pool), A3/A6↔DP-0005(공유 캐시·memoization) 일관 결정 필요.

## 출처 (리서치 근거)
- R-01 [Azure Competing Consumers](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers) · [Claim-Check](https://learn.microsoft.com/en-us/azure/architecture/patterns/claim-check)
- R-02 [Azure Pipes and Filters](https://learn.microsoft.com/en-us/azure/architecture/patterns/pipes-and-filters) · [Knative/Serverless](https://www.cloudraft.io/blog/building-serverless-functions-on-kubernetes-using-knative)
- R-03 [Temporal Durable Execution](https://temporal.io/blog/temporal-replaces-state-machines-for-distributed-applications) · [Saga Orchestration vs Choreography](https://temporal.io/blog/to-choreograph-or-orchestrate-your-saga-that-is-the-question)
- R-04 [Microkernel Architecture](https://softwaresystemdesign.com/software-architecture-design/architectural-patterns/microkernel-architecture/)
- R-05 [Kubernetes Pod Affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) · [Local PersistentVolume](https://kubernetes.io/docs/concepts/storage/volumes/#local) · [HDFS Data Locality](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
