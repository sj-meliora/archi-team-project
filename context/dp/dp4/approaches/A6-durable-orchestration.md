# A6 (6안) Durable Orchestration — Event-Sourced Workflow Engine

> category: DP-approach | for: DP-0004 | status: 발굴·평가완료 | updated: 2026-06-20
> 근거 리서치: R-03 | drives: QA-08(주), QA-12, QA-10, QA-01 | realizes: FR-0001

## 구조
워크플로 엔진(Temporal / Argo Workflows류)이 각 노드 작업(Converter→Optimizer→Quantizer→Compiler)을 **durable event log**로 기록한다. crash·재시작 시 event sourcing **재생**으로 실패 직전 상태를 정확히 복원하고, 노드 실패 시 **자동 retry + Saga 보상 트랜잭션**으로 해당 노드만 복구한다. workflow를 코드/선언(YAML)으로 명시(Workflow-as-code)해 흐름을 1급 객체로 다룬다. 실제 노드 컴퓨팅은 엔진 worker(또는 A5 ephemeral Job)가 수행.

## 근거 tactic/pattern
- **Durable Execution**, **Event Sourcing(immutable log replay)**, **Saga(Orchestration)**, **Compensating Transaction**, **Automatic retry**, **Idempotent activity**.

## 기존 1·2안과의 차별점
- 1·2안·A3·A5는 노드의 **실행/분배 메커니즘**을 결정 → A6는 **실행 상태의 내구성·복구**를 1급 관심사로 둠(직교적 보강). FR-0001의 "노드 단위 재시도·복구"를 엔진 차원에서 구조적으로 보장. A3(Choreography)와 대비되는 **Orchestration 축**.

## QA별 장점 / 단점
- **[Reliability-Workflow QA-08] ★★★ (주 강점)**: durable replay로 crash 후 무손실 복구, 실패 노드만 자동 retry/보상 → 타 Workflow 무영향(중단≤1% 직접 겨냥). immutable event log로 SPOF 완화.
- **[Maintainability QA-12] ★★★**: workflow를 코드/선언으로 명시 → 흐름 가시성↑, 노드 추가·변경·재정렬이 명시적이라 Change Impact 추적 용이.
- **[Performance-E2E QA-10] ★★☆**: ⚠️ event log 영속화·엔진 hop 오버헤드. 단 장기 실행 timeout 부재로 대형 컴파일에 안정. 전달 오버헤드 ≤5% 검증 필요.
- **[Scalability QA-01] ★★☆**: 엔진 worker는 수평 확장 가능하나 ⚠️ **중앙 오케스트레이터/이벤트 스토어가 확장 한계점**(DP-0002 TP-2 병목과 연계) → 샤딩/네임스페이스 분할 필요.

## Trade-off 별점
| Performance | Scalability | Reliability-WF | Maintainability |
|:---:|:---:|:---:|:---:|
| ★★☆ | ★★☆ | ★★★ | ★★★ |

## mini-ATAM
- **SP**: 이벤트 스토어 영속화 빈도가 QA-10(오버헤드≤5%)에 민감. 오케스트레이터 샤딩 여부가 QA-01에 민감.
- **Risk**: 중앙 오케스트레이터/이벤트 스토어 확장 한계 → 모델 폭증 시 병목(QA-01 위협). event log 비대화로 replay 비용↑.
- **Non-Risk**: C-01(Docker) — Temporal/Argo 모두 컨테이너·K8s 네이티브 구동.

## 인접 DP 정합
- ⚠️ **DP-0002(Agent Hierarchy) 오케스트레이터 논의와 직접 정합 필요** — A6의 워크플로 오케스트레이터를 DP-0002의 계층 제어와 일치시키거나 중복 방지. 엔진 worker로 **A5(ephemeral Job)** 를 쓰면 Reliability(A6)+Scalability(A5) 결합. DP-0005 캐시는 activity 결과 memoization으로 결합.
