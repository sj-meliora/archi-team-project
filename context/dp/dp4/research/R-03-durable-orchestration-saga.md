# R-03 리서치 — Durable Execution / Event Sourcing / Saga 기반 워크플로 엔진

> category: DP-research | for: DP-0004 | updated: 2026-06-20
> 목적: 실행 "메커니즘"이 아니라 **실행 상태의 내구성·복구**를 1급 관심사로 둔 패턴 조사. FR-0001(노드 격리·재시도·복구) 정조준.

## 조사한 패턴

### 1) Durable Execution (내구 실행) + Event Sourcing
- **정의**: 워크플로의 모든 액션·결정을 **immutable event log**로 durable하게 기록. crash/재시작 시 event를 **재생(replay)** 해 실패 직전 상태로 정확히 복원. 동기 코드처럼 작성하지만 상태가 영속.
- **효과**: 장기 실행(일·주·월) workflow도 timeout 없이 지속, 자동 retry, crash 무손실 복구. event sourcing으로 각 상태 변화가 immutable event → **단일 장애점 회피**(중앙 엔진이라도 상태는 로그에 분산).
- 대표 구현: **Temporal**(durable app workflow, Coinbase/Netflix), **Argo Workflows**(K8s-native, YAML/Python, CI/CD·데이터 파이프라인에 강점).
- 출처: [Temporal — Beyond State Machines](https://temporal.io/blog/temporal-replaces-state-machines-for-distributed-applications), [Temporal vs Argo (xgrid)](https://www.xgrid.co/resources/temporal-vs-argo-workflows-architecture-comparison/)

### 2) Saga (Orchestration vs Choreography)
- **Saga**: 큰 분산 트랜잭션을 서비스별 sub-transaction으로 쪼개고, 한 단계 실패 시 **보상 트랜잭션(compensating action)** 으로 일관성 회복.
- **Orchestration**: 중앙 오케스트레이터가 순서대로 명령·응답 대기·보상 → **중앙 제어·흐름 가시성·디버깅 용이**(복잡한 분기·조건에 유리). 단 오케스트레이터가 관심사.
- **Choreography**: 각 서비스가 이벤트로 자율 협력 → 결합도↓·확장성↑, 단 **가시성·디버깅 어려움**, message broker가 SPOF.
- 본 과제 함의: A3(큐 이벤트 구동)는 Choreography 축, **A6(워크플로 엔진)는 Orchestration 축** — 서로 대비되는 보강.
- 출처: [Saga Orchestration vs Choreography (Temporal)](https://temporal.io/blog/to-choreograph-or-orchestrate-your-saga-that-is-the-question)

## DP-0004에의 함의
- 1·2안·A3·A5는 "노드를 어디서/어떻게 분배·실행하나"에 집중 → **A6는 "실행 상태가 죽지 않고 복구되나"** 를 구조적으로 보장.
- FR-0001의 "노드 단위 재시도·복구"를 엔진 차원에서 1급으로 제공, 실패 노드만 보상·재실행 → QA-08(타 Workflow 무영향) 직접 강화.
- ⚠️ 단 Orchestration 중심이라 **DP-0002(Agent Hierarchy)의 오케스트레이터 병목/SPOF 논의(TP-2)** 와 정합 필요. event sourcing이 SPOF를 완화하나 중앙 이벤트 스토어가 확장 한계점이 될 수 있음.

→ 도출 대안: **[A6] Durable Orchestration (Event-Sourced Workflow Engine)**.
