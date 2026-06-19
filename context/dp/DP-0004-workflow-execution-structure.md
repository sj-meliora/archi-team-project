# DP-0004 Workflow 실행 구조

> category: DP | status: 결정대기 | source: pptx p.32 | updated: 2026-06-20
> drives: QA-0002(Scalability), QA-0006(Reliability-Workflow), QA-0008(Performance-E2E), QA-0010(Maintainability)
> realizes: FR-0001, FR-0002 | constrained-by: C-0001(Docker)
> note: ⚠️ 원본 헤더가 DP-0002 라벨(Hierarchical/Decentralized) 복붙 오류 — 내용 기준 라벨로 교정 (OI-3)

## 결정 포인트
노드 작업을 어떤 실행 구조로 처리할 것인가 — **타입별 공유 서버 풀 vs 노드당 독립 인스턴스**.

## 후보 대안

### 1안. 타입별 전용 서버 풀
- **구조**: 노드 작업을 타입별 서버 풀로 라우팅 + 오브젝트 스토리지로 중간 Artifact 전달, 풀별 독립 scale out.
- **tactic/pattern**: Shared resource pool, Staging, Object storage.
- **장점**: [Scalability] 노드 타입별 독립 scale out, 자원 utilization 유리 / [Performance] 사전 Staging으로 오버헤드 은닉.
- **단점**: [Reliability] 노드 타입 장애가 해당 타입 쓰는 모든 Workflow로 전파 / [Maintainability] 풀·스토리지·Staging 관리 복잡.

### 2안. 노드당 Workflow 인스턴스
- **구조**: 노드당 다중 Workflow 인스턴스 + 로컬 스토리지, 노드 단위 격리·scale out.
- **tactic/pattern**: Instance-per-node, Bulkhead, Local storage.
- **장점**: [Reliability] 노드 격리로 장애 독립성 구조적 보장 / [Performance] 네트워크 전달 없음 → 오버헤드 최소 / [Maintainability] 인프라 단순.
- **단점**: [Scalability] Scale 단위가 Workflow 전체로 고정 → 병목 시 전체 복제로 자원 낭비.

## Trade-off 매트릭스
| 대안 | Performance | Scalability | Reliability | Maintainability |
|---|:---:|:---:|:---:|:---:|
| 1안 타입별 서버 풀 | ★★☆ | ★★★ | ★★☆ | ★☆☆ |
| 2안 노드당 인스턴스 | ★★★ | ★★☆ | ★★★ | ★★★ |

## ATAM 분석
### 민감점 (Sensitivity Points)
- **SP-1**: 중간 Artifact 전달 방식(오브젝트 스토리지 vs 로컬)이 Performance(QA-0008, 전달 오버헤드 ≤5%)에 민감.
- **SP-2**: 장애 격리 단위(타입 풀 vs 노드)가 Reliability(QA-0006, 타 Workflow 중단 ≤1%)에 민감.

### 교환점 (Tradeoff Points)
- **TP-1 (Scalability ↔ Reliability/Maintainability)**: 공유 풀(1안)은 확장성·활용률↑·장애전파 위험↑, 노드 격리(2안)는 반대.

### 위험 (Risks)
- **R-1**: 1안은 타입 풀 장애가 해당 타입 모든 Workflow로 전파 → QA-0006(중단≤1%) 미달 위험.
- **R-2**: 2안은 Workflow 단위 복제로 자원 낭비 → QA-0002(활용률≥70%) 미달 위험.

### 비위험 (Non-Risks)
- **NR-1**: 두 안 모두 C-0001(Docker) 컨테이너 기반 구동 가능.

## 결정 / 근거
- (미정.) 권고: QA-0006(Reliability)이 격리 요구를 강제하면 **2안 기반**, 모델 폭증으로 활용률이 관건이면 1안. → DP-0001·DP-0005와 함께 일관 결정 필요.
