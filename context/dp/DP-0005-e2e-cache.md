# DP-0005 E2E 개발시간 최적화 (캐시 전략)

> category: DP | status: 결정대기 | source: pptx p.33 | updated: 2026-06-20
> drives: QA-10(Performance-E2E), QA-08(Reliability-Workflow), QA-11(Reliability-Agent), QA-12(Maintainability)
> realizes: FR-0002

## 결정 포인트
E2E 개발시간 단축을 위해 캐시를 **Workflow별 독립 vs Workflow 간 공유** 중 어떻게 둘 것인가.

## 후보 대안

### 1안. Workflow별 로컬 캐시
- **구조**: Workflow마다 독립 Cache.
- **tactic/pattern**: Local cache, Bulkhead.
- **장점**: [Reliability-Workflow] 한 Workflow 오류가 타에 무영향 / [Maintainability] 공유 인프라 없이 단순.
- **단점**: [Performance] 유사 설정 타 Workflow 결과 재사용 불가 / [Reliability-Agent] Workflow 간 결과 편차.

### 2안. Workflow 간 공유 캐시
- **구조**: 중앙 Shared Cache 공유.
- **tactic/pattern**: Shared cache, Memoization.
- **장점**: [Performance] 유사 Workflow 간 캐시 공유로 E2E 단축 / [Reliability-Agent] Workflow 간 Agent 판단 일관성 보장.
- **단점**: [Reliability-Workflow] 공유 캐시 오류가 타 Workflow로 전파 / [Maintainability] 중앙 캐시 추가, 모듈 교체 범위 판단 필요.

## Trade-off 매트릭스
| 대안 | Performance | Reliability-Workflow | Reliability-Agent | Maintainability |
|---|:---:|:---:|:---:|:---:|
| 1안 로컬 캐시 | ★★☆ | ★★★ | ★★☆ | ★★★ |
| 2안 공유 캐시 | ★★★ | ★★☆ | ★★★ | ★★☆ |

## ATAM 분석
### 민감점 (Sensitivity Points)
- **SP-1**: 캐시 공유 범위가 Performance(QA-10, E2E)와 Reliability-Workflow(QA-08)에 동시 민감.
- **SP-2**: 캐시 공유가 Agent 판단 일관성(QA-11, 재현율≥80%)에 민감.

### 교환점 (Tradeoff Points)
- **TP-1 (Performance/Agent일관성 ↔ Workflow독립성)**: 공유(2안)는 재사용·일관성↑·장애전파 위험↑, 로컬(1안)은 반대. 본 DP의 핵심 교환점.

### 위험 (Risks)
- **R-1**: 2안은 공유 캐시 오염이 다수 Workflow로 전파 → QA-08(중단≤1%) 미달 위험.
- **R-2**: 1안은 재사용 불가로 QA-10(E2E) 개선폭 제한.

### 비위험 (Non-Risks)
- **NR-1**: 두 안 모두 FR-0002(Artifact 저장·전달) 인프라 위에 구현 가능.

## 결정 / 근거
- (미정.) 권고: Performance·Agent 일관성을 위해 **2안 기반**, 전파 위험(R-1)은 캐시 무효화·검증·읽기전용 계층화로 완화. DP-0004와 격리 정책 정합 필요.
