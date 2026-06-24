# DP-0001 Workflow – Agent 매핑

> category: DP | status: 결정대기 | source: pptx p.29 | updated: 2026-06-20
> drives: QA-09(Performance-Agent), QA-02(Availability), QA-01(Scalability), QA-05(Efficiency)
> realizes: FR-0001

## 결정 포인트
Workflow 노드에 Agent를 어떻게 배치할 것인가 — **고정 배치(즉시성·격리) vs 동적 할당(확장·최적화)**.

## 후보 대안

### 1안. Per-Node Agent
- **구조**: 노드별 전용 Agent 고정 배치.
- **tactic/pattern**: Static binding, Bulkhead(노드별 격리).
- **장점**: [Performance] 선택 없이 즉시 실행 / [Availability] 고정 구조로 장애 격리·운영 안정.
- **단점** *(원본 복붙 오류 교정 — OI-2)*: [Scalability] 고정 배치라 노드 수 증가/변동에 확장 경직 / [Efficiency] 노드별 전용 Agent 상시 점유 → 유휴 자원·토큰 낭비.

### 2안. Dynamic Agent Pool
- **구조**: Agent Router가 Pool에서 동적 할당.
- **tactic/pattern**: Resource pooling, Dynamic routing.
- **장점**: [Scalability] 동적 할당으로 확장 용이 / [Efficiency] 작업별 최적 Agent 선택.
- **단점**: [Performance] Routing 지연 / [Availability] 동적 경로로 운영 복잡성 증가.

## Trade-off 매트릭스
| 대안 | Efficiency | Scalability | Performance | Availability |
|---|:---:|:---:|:---:|:---:|
| 1안 Per-Node | ★★☆ | ★★☆ | ★★★ | ★★★ |
| 2안 Dynamic Pool | ★★★ | ★★★ | ★★☆ | ★★☆ |

## ATAM 분석
### 민감점 (Sensitivity Points)
- **SP-1**: Agent Router의 routing 지연이 Performance(QA-09)에 민감 (2안).
- **SP-2**: Agent 배치 정적/동적 여부가 Scalability(QA-01, 자원활용률≥70%)에 민감.

### 교환점 (Tradeoff Points)
- **TP-1 (Performance ↔ Scalability/Efficiency)**: 고정 배치는 즉시성↑·확장성↓, 동적 풀은 그 반대. 본 DP의 핵심 교환점.

### 위험 (Risks)
- **R-1**: 1안은 모델·노드 폭증(배경 워크로드 축) 시 자원활용률 70% 미달 위험.
- **R-2**: 2안의 routing 지연이 QA-09(수행시간 단축) 효과를 잠식할 위험.

### 비위험 (Non-Risks)
- **NR-1**: 두 안 모두 노드 단위 재시도/복구(FR-0001)는 지원 가능.

## 결정 / 근거
- (미정.) 권고: 모델 수 폭증 배경상 Scalability·Efficiency 비중이 커 **2안 기반**, routing 지연(R-2)은 캐싱/사전 워밍으로 완화 검토.
