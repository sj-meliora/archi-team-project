# DP-0002 Agent Hierarchy

> category: DP | status: 결정대기 | source: pptx p.30 | updated: 2026-06-20
> drives: QA-03(Controllability)↑, QA-02(Availability), QA-10(Performance-E2E), QA-01(Scalability)
> realizes: FR-0004

## 결정 포인트
Agent들을 어떤 위상(topology)으로 구성할 것인가 — **중앙 제어성 vs 가용성** 트레이드오프.

## 후보 대안

### 1안. Hierarchical Multi-Agent
- **구조**: Orchestrator 중심 (Workflow Engine 하위).
- **tactic/pattern**: Orchestration, Centralized policy, HITL gate.
- **장점**: [Controllability] 단일 지점 정책·HITL 승인 / [Scalability] Workflow별 인스턴스 생성.
- **단점**: [Availability] Orchestrator 장애 = 전체 장애(SPOF) / [Performance] Orchestrator 경유 병목.

### 2안. Fully Decentralized Multi-Agent
- **구조**: Agent 간 직접 통신 (Orchestrator 없음).
- **tactic/pattern**: Peer-to-peer, Choreography.
- **장점**: [Availability] 단일 장애점 없음, 장애 무전파 / [Performance] 직접 통신, 오버헤드 적음.
- **단점**: [Controllability] 정책 분산 → 추가 비용(HITL·토큰) / [Scalability] 결합도 높아 구성 변경 시 호출관계 수정.

### 3안 (신규 후보). Hierarchical + Standby Orchestrator
- **구조**: 1안 + Orchestrator를 Active-Passive 이중화, 상태 외부화.
- **근거 tactic**: [Availability] Redundancy(Active-Passive) + Heartbeat + State resync.
- **의도**: 1안의 Controllability ★★★를 유지하면서 SPOF를 완화.
- **미검증 trade-off**: 페일오버 중 일관성(QA-11), 대기 인스턴스 자원, 페일오버 시간 vs MTTR<1분(QA-02).
- → 상세 논의: `_backlog.md` BL-1.

## Trade-off 매트릭스
| 대안 | Controllability | Scalability | Availability | Performance(E2E) |
|---|:---:|:---:|:---:|:---:|
| 1안 Hierarchical | ★★★ | ★★☆ | ★★☆ | ★★☆ |
| 2안 Decentralized | ★★☆ | ★★☆ | ★★★ | ★★★ |
| 3안 H+Standby (추정) | ★★★ | ★★☆ | ★★★ | ★★☆ |

## ATAM 분석
> 이 결정에 대한 민감점·교환점·위험을 문서 내부에서 분석한다.

### 민감점 (Sensitivity Points)
- **SP-1**: Orchestrator의 가용성이 시스템 전체 Availability(QA-02, MTTR<1분)를 좌우 → 1안에서 특히 민감.
- **SP-2**: 정책 적용 지점의 집중도가 Controllability(QA-03, 중단 ≤5초)를 좌우.

### 교환점 (Tradeoff Points)
- **TP-1 (Controllability ↔ Availability)**: 제어를 한 점에 모을수록(1안) Controllability↑·Availability↓. 분산할수록(2안) 반대. → 본 DP의 핵심 교환점.
- **TP-2 (Performance ↔ Controllability)**: Orchestrator 경유는 HITL·정책 적용을 쉽게 하지만 E2E 병목.

### 위험 (Risks)
- **R-1**: 1안 채택 시 Orchestrator SPOF가 QA-02(MTTR<1분) 미달 위험.
- **R-2**: 2안 채택 시 분산 정책으로 QA-03(중단 ≤5초) 미달 + 토큰 비용 증가(QA-05) 위험.

### 비위험 (Non-Risks)
- **NR-1**: Workflow별 인스턴스 생성으로 Scalability는 1·2안 모두 수용 가능(★★☆ 동급).

## 결정 / 근거
- (미정 — open-issues 아님. FR-0004의 제어·관측 우선순위가 높으면 1안/3안, 가용성 최우선이면 2안.)
- 권고: Controllability가 H 중요도(QA-03)이고 C-0002(기존 시스템 무영향)가 강하므로 **1안 기반**, R-1은 **3안(Standby)**으로 완화 검토.
