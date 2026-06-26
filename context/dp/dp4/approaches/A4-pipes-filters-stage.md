# A4 (4안) Pipes-and-Filters Stateless Stage — 구조적 단계 분해

> category: DP-approach | for: DP-0004 | status: 발굴·평가완료 | updated: 2026-06-20
> 근거 리서치: R-02 | drives: QA-12(주), QA-01, QA-08, QA-10

## 구조
파이프라인 4단계(IR Converter / Graph Optimizer / Quantizer / Compiler)를 각각 **독립·무상태 filter**로 구현하고, **표준 I/O 스키마**(예: ModelArtifact + ConfigManifest)로 pipe(큐 또는 스토리지)를 통해 연결한다. filter는 서로를 모르고 입·출력 스키마만 안다 → 단계 **재정렬·재사용·삽입/제거**가 자유롭고, 느린 filter만 병렬 인스턴스로 확장한다. 상태는 외부 스토어(claim-check)에 두어 filter를 무상태로 유지한다.

## 근거 tactic/pattern
- **Pipes and Filters**, **Stateless component**, **Standardized schema(compositional reuse)**, **Idempotent filter**, (분산 트랜잭션 대안으로 **Compensating Transaction**).

## 기존 1·2안과의 차별점
- 1·2안·A3는 "노드를 **어디서** 실행하나"를 결정 → A4는 "실행 단위를 **어떻게 구조적으로 분해**하나"를 결정하는 **직교 축**. 즉 A4는 1안/2안/A3 어느 실행 기반 위에도 얹을 수 있는 **구조 스타일**이며, 특히 Maintainability(QA-12)와 조합 폭발 대응을 직접 겨냥.

## QA별 장점 / 단점
- **[Maintainability QA-12] ★★★ (주 강점)**: 단일 책임 filter + 표준 스키마 → 한 단계 구현 교체가 그 filter에 국소화(Change Impact Scope ≤2 직접 달성), 새 단계 삽입·재정렬 용이.
- **[Scalability QA-01] ★★★**: filter별 독립 배포·병렬화, 병목 filter만 선택적 확장 → 자원 활용률↑.
- **[Reliability-Workflow QA-08] ★★☆**: 단일 책임으로 장애 국소화. ⚠️ 단 전형적 구현이 **monolithic pipeline** — filter/pipe 하나 실패가 파이프라인 전체 실패로 전파 가능 → pipe 데이터 유실 방지·멱등 필수.
- **[Performance-E2E QA-10] ★★☆**: 스트림 I/O 시 단계 중첩 실행으로 처리량↑. ⚠️ 단 단계 수만큼 hop + 외부 상태 load/persist 오버헤드(≤5% 목표 검증 필요).

## Trade-off 별점
| Performance | Scalability | Reliability-WF | Maintainability |
|:---:|:---:|:---:|:---:|
| ★★☆ | ★★★ | ★★☆ | ★★★ |

## mini-ATAM
- **SP**: filter 무상태성 + pipe 유실방지 인프라가 QA-08·QA-10에 민감. 표준 스키마 안정성이 QA-12(재정렬 효과)에 민감.
- **Risk**: monolithic 특성상 단계 fault가 전체 파이프라인 중단(QA-08 중단≤1% 위협) → 큐 기반 pipe + DLQ(A3와 결합)로 완화. 외부 상태 왕복이 QA-10 잠식.
- **Non-Risk**: C-01(Docker) — filter를 컨테이너로 배포 가능. C-02(이식성)도 표준 스키마로 유리.

## 인접 DP 정합
- **A3(메시지 큐)를 pipe 구현체로 채택하면 A4의 monolithic 위험을 큐 버퍼+DLQ로 상쇄** → A3+A4 결합이 강력. DP-0005 캐시는 filter 입력 스키마 키로 memoization 가능.
