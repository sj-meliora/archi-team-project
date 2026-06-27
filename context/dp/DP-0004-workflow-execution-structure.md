# DP-0004 Workflow 실행 구조

> category: DP | status: 결정대기(A5 vs A8로 수렴) | source: pptx p.32 | updated: 2026-06-22
> drives: QA-01(Scalability), QA-08(Reliability-Workflow), QA-10(Performance-E2E), QA-12(Maintainability)
> realizes: FR-0001, FR-0002 | constrained-by: C-01(Docker)
> 보강작업: `dp4/`(INDEX·decision-axes·approaches A3~A8·evaluation·review)
> note: ⚠️ 원본 헤더가 DP-01 라벨(Hierarchical/Decentralized) 복붙 오류 — 내용 기준 라벨로 교정 (OI-3)

## 결정 포인트
파이프라인 노드 작업(IR Converter → Graph Optimizer → Quantizer → Compiler)을 **어떤 실행 구조**로 처리할 것인가.

## 결정 구도의 진화 — 1안·2안 → A5 vs A8
원래 결정은 **1안(타입별 공유 서버 풀) vs 2안(노드당 인스턴스)** 이었다. `dp4/` 보강작업(패턴 기반 대안 A3~A8 발굴)에서 축 A(실행위치/자원 모델)를 **2개 직교 손잡이**로 분해해 네 대안을 2x2에 고정했다(`dp4/decision-axes.md`).

- **축 ① 자원 수명**: 상주 풀(standing) ↔ 일회용(ephemeral, scale-to-zero)
- **축 ② 데이터-컴퓨트 배치**: 데이터를 컴퓨트로(원격, claim-check) ↔ 컴퓨트를 데이터로(로컬, data-affinity)

| | **원격** (전달세금↑, 복구 유리) | **로컬** (전달세금↓, 복구 불리) |
|---|---|---|
| **상주 풀** | **1안** 타입별 공유 서버 풀 | **2안** co-located 상주 인스턴스 |
| **일회용** | **A5** ephemeral, 클러스터 자유 배치 | **A8** ephemeral + 데이터 노드 핀 |

각 계열에서 **일회용(A5·A8)이 상주(1안·2안)의 유휴 낭비·격리 약점을 고친 진화형**이다 → 1안은 A5에, 2안은 A8에 지배되어 **기준선(null)으로 격하**. 따라서 **최종 대결은 A5 vs A8** — 둘 다 일회용이고, **전달 배치(원격↔로컬)** 한 축에서만 갈린다.

## 두 결선 대안

### A5. Serverless / Ephemeral Compute-per-Node (원격 계열)
- **구조**: 노드 작업 **호출마다** ephemeral 컨테이너(K8s Job / Knative / FaaS)를 띄워 1건 처리 후 종료. 중간 Artifact(20GB+)는 오브젝트 스토리지(claim-check)로 전달, 컴퓨트는 클러스터 어디든 자유 배치. (상세: `dp4/approaches/A5-*`)
- **tactic/pattern**: FaaS, Ephemeral instance, Scale-to-zero, Bulkhead(invocation 격리), Claim-Check, Built-in retry.
- **장점**: [Scalability] 호출 단위 near-infinite auto-scale + scale-to-zero → 과프로비저닝 0 / [Reliability] invocation별 독립 sandbox·자동 retry + **오브젝트 스토리지가 노드 사망에도 산출물 보존 → 복구 유리** / [Maintainability] Job 단위 독립 배포.
- **단점**: [Performance] ⚠️ **claim-check 20GB 왕복 전달세금** + 대형 이미지 cold-start → E2E budget 잠식(pre-warm/min-instance 필요).

### A8. Co-located Ephemeral Stage (로컬 계열)
- **구조**: 4단계를 **데이터가 있는 노드에 핀(data-affinity)** 으로 박아 실행. 수명 모델은 A5와 동일(일회용 컨테이너)이나, 스케줄러가 **이전 단계가 20GB를 남긴 그 노드** 위에 배치 → 단계 간 전달은 **노드 로컬 볼륨/스트리밍**. 노드 내 단계별 worker를 비대칭(병목 단계만↑)으로 두고 로컬 용량 초과 시에만 원격 spill(locality-first). (상세: `dp4/approaches/A8-*`)
- **tactic/pattern**: Data-Locality Scheduling, Co-location/Pod Affinity/Volume Locality, Pipeline Parallelism(단계 비대칭 worker), Bounded Buffer, Ephemeral instance, Bulkhead(단계 격리).
- **장점**: [Performance] **claim-check 20GB 왕복 회피**(2안의 유일 강점 계승) → E2E ★★★ / [Scalability] 병목 단계 worker만 노드 내 탄력 확장(단계 입도) → 유휴 제거.
- **단점**: [Reliability] ⚠️ **로컬 디스크 내구성 약함** — 노드 사망 시 그 위 산출물 유실 → 해당 단계부터 재실행(복구 불리) / [Performance] 노드 footprint 초과 spill 시 원격 전달로 degrade(A5에 수렴).

## Trade-off 매트릭스
| 대안 | Performance(E2E) | Scalability | Reliability-WF | Maintainability |
|---|:---:|:---:|:---:|:---:|
| 1안 타입별 서버 풀 (기준선) | ★★☆ | ★★★ | ★★☆ | ★☆☆ |
| 2안 노드당 인스턴스 (기준선) | ★★★ | ★★☆ | ★★★ | ★★★ |
| **A5 ephemeral·원격** | ★★☆ | ★★★ | ★★★ | ★★☆ |
| **A8 ephemeral·로컬** | ★★★ | ★★★ | ★★☆ | ★★☆ |

> A5·A8 모두 일회용이라 Scalability ★★★(과프로비저닝 0)·기준선 대비 활용률 우위. 차이는 가로축에 집중 — **A8은 전달 우위(Perf↑)·복구 약(Rel↓)**, **A5는 복구 우위(Rel↑)·전달 약(Perf↓)**.

## ATAM 분석 (A5 vs A8 초점)

### 민감점 (Sensitivity Points)
- **SP-1 (전달 배치 → Performance)**: 중간 Artifact 전달 방식(claim-check 원격 vs 로컬 스트리밍)이 QA-10(전달 오버헤드 ≤5%)에 강하게 민감. **A5 vs A8을 가르는 핵심 손잡이**.
- **SP-2 (산출물 내구성 → Reliability)**: 산출물 저장 위치(오브젝트 스토리지 vs 노드 로컬 디스크)가 QA-08(노드 사망 시 재실행 범위)에 민감.
- **SP-3 (cold-start)**: 일회용 공통 약점. 이미지 크기·pre-warm/min-instance 정책이 QA-10·QA-09에 민감(A5·A8 공통이나 A5가 더 노출).
- **SP-4 (노드 footprint, A8 고유)**: 모델당 파이프라인이 단일 노드 용량에 들어가는지가 A8의 로컬리티 유지 vs spill에 민감.

### 교환점 (Tradeoff Points)
- **TP-1 (Performance ↔ Reliability)**: 가로축이 곧 교환점. **A8 = 로컬 전달로 전달세금 회피(Perf↑) ↔ 로컬 디스크 유실로 복구 불리(Rel↓)**, **A5 = claim-check 전달세금(Perf↓) ↔ 스토리지 보존으로 복구 유리(Rel↑)**.
- **TP-2 (단순성 ↔ 로컬리티 제어)**: A5는 배치 자유로 스케줄러 단순, A8은 data-affinity·로컬 볼륨 수명관리로 Maintainability 비용 추가.

### 위험 (Risks)
- **R-1 (A5)**: 대형 컴파일러 이미지 cold-start + 20GB claim-check 왕복으로 E2E ≤5% 전달 budget(QA-10) 미달 위험 → pre-warm pool·이미지 슬림화로 완화.
- **R-2 (A8)**: 모델 폭증으로 노드 용량 초과 → **로컬리티 붕괴, 원격 전달 degrade(A8 강점 소멸 → A5에 수렴)**. 로컬 디스크 유실 시 재실행 비용↑ → 복제/체크포인트 정책 필요.
- **R-3 (공통)**: 일회용 scale-to-zero 후 동시 폭증 시 throttling → min-instance 하한 필요.

### 비위험 (Non-Risks)
- **NR-1**: A5·A8 모두 C-01(Docker)·C-02(이식성) 충족 — K8s Job / Knative(+pod affinity·local volume) 컨테이너 기반 구동.

## 결정을 가르는 단일 질문
> **`(4단계 × 20GB 왕복) ÷ 스토리지 대역폭`이 E2E의 5% budget(QA-10) 안에 드는가?**
- **든다 → A5** (전달세금 감내 가능 → 단순·복구 유리 채택).
- **넘는다 → A8** (전달세금 회피 필요 → 로컬 전달, locality-first 채택).
- 이 산식은 **실측·노드 사양 의존 → 팀 검증 대상**. 본 결정은 *구조*를 고정하고 수치는 측정으로 채운다.

## 결정 / 근거
- **수렴 결론**: 후보를 **A5(일회용·원격) vs A8(일회용·로컬)** 둘로 압축. 1안·2안은 비교 기준선으로만 유지(각 진화형에 지배).
- **택일 기준**: 위 5% 산식(SP-1/TP-1)이 최종 택일을 결정. 부차적으로 — 노드가 모델 파이프라인을 **못 담으면 A8 보류**(R-2 spill), cold-start를 목표 이하로 **못 누르면 A5 보류**(R-1).
- **직교 보강축은 동시 채택 아님**(상세 `dp4/evaluation.md`): A4(무상태 분해)는 선택안 위 설계 규율로만, A3↔A6(제어평면)은 **DP-01와 동시 단일화**(이중 제어평면 금지), A7(변종)은 변종 수 임계 초과 시에만.
- **인접 DP 정합**: A5·A8 모두 구 DP-0001 2안(Dynamic Agent Pool)과 정합. A8은 DP-0005 캐시를 노드 로컬 볼륨 계층과 결합. → 구 DP-0001·DP-0005와 격리·확장 정책 일관 결정 필요.
