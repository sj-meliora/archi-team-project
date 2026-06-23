# DP-0003 Agent 외부 시스템 안정성 보장

> category: DP | status: 결정대기 | source: pptx p.31 | updated: 2026-06-20
> drives: QA-03(Controllability), QA-02(Availability), QA-04(Observability), QA-05(Efficiency)
> realizes: FR-0004 | constrained-by: C-0002(기존 시스템 무영향)

## 결정 포인트
Agent가 외부 시스템(Jira·빌드서버 등)에 영향을 주지 않도록 **어떻게 안정성을 보장**할 것인가.

## 후보 대안

### 1안. 사전 권한 체크
- **구조**: 권한 체크 게이트(정책·allowlist), 허용만 통과.
- **tactic/pattern**: Authorization gate, Allowlist, Policy enforcement point.
- **장점**: [Controllability] 허용 범위 직접 제어 / [Efficiency] 토큰 미소모·저오버헤드.
- **단점**: [Availability] 미정의 액션·우회 경로는 차단 불가.

### 2안. 격리 환경 수행
- **구조**: 격리 경계 + 브로커/프록시 경유, 직접 경로 차단.
- **tactic/pattern**: Sandbox, Broker/Proxy, Bulkhead.
- **장점**: [Availability] 외부 직접 접근 차단을 구조적으로 보장 / [Efficiency] 격리는 토큰 무관.
- **단점**: [Performance] 정상 작업도 브로커 경유 / [Scalability] 격리 환경 수백 개 → 자원 점유.

### 3안. 실시간 모니터링
- **구조**: Agent↔외부 직접 + 실시간 모니터가 관측·차단·롤백.
- **tactic/pattern**: Runtime monitoring, Anomaly detection, Rollback.
- **장점**: [Observability] 실행·판단 근거 추적 / [Availability] 빠른 탐지·복구로 MTTR 단축.
- **단점**: [Availability] 사후 탐지(예방 불가) / [Efficiency] LLM 기반 탐지 시 토큰 폭증.

## Trade-off 매트릭스
| 대안 | Availability | Performance | Efficiency | Controllability |
|---|:---:|:---:|:---:|:---:|
| 1안 사전 권한 체크 | ★★☆ | ★★★ | ★★★ | ★★★ |
| 2안 격리 환경 | ★★★ | ★★☆ | ★★★ | ★★☆ |
| 3안 실시간 모니터링 | ★☆☆ | ★★☆ | ★☆☆ | ★★★ |

## ATAM 분석
### 민감점 (Sensitivity Points)
- **SP-1**: 차단 시점(사전 vs 사후)이 Availability(QA-02)·C-0002 보장 강도에 민감.
- **SP-2**: 탐지 방식(규칙 vs LLM)이 Efficiency(QA-05, 토큰)에 민감 (3안).

### 교환점 (Tradeoff Points)
- **TP-1 (Availability ↔ Performance)**: 격리(2안)는 구조적 안전성↑·정상경로 오버헤드↑.
- **TP-2 (예방력 ↔ Observability)**: 사전 차단(1·2안)은 예방↑·관측↓, 실시간 모니터(3안)는 관측↑·예방↓.

### 위험 (Risks)
- **R-1**: 1안 단독은 미정의 액션 우회로 C-0002 위반 위험.
- **R-2**: 3안 단독은 사후 탐지라 이미 발생한 외부 영향 차단 불가(C-0002 위반 위험) + 토큰 폭증.

### 비위험 (Non-Risks)
- **NR-1**: 1안의 저오버헤드는 QA-05(토큰) 관점에서 안전.

## 결정 / 근거
- (미정.) 권고: C-0002가 강제 제약이므로 **1안(사전 권한) + 2안(격리)** 조합으로 예방을 구조화하고, **3안(모니터링)**은 Observability(QA-04) 보강용으로 규칙 기반 한정 적용.
