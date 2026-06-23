# Counsel: QA-10 Maintainability — 모듈 교체 용이성

> refs-review: round-01/review/QA-10-maintainability.md · report.md(C4)
> seats: 발의 Seat 2(수석 아키텍트) · 합의 consensus (Seat 1 prompt/model 축, Seat 3 워크플로우 버저닝)
> stance: 채택 권장 (건강 — tail SLI 전환 + agentic 유지보수축 추가)

## Reviewer 지적 요약
- **건강한 QA** — 모듈성(낮은 결합) 단일 관심사 명확, CIS 측정 가능 (렌즈2).
- **평균의 함정**: `평균 CIS ≤ 2`는 tail 은닉(한 변경이 12개 건드려도 평균에 묻힘) → **p95/max** (렌즈2).
- **agentic 유지보수축 누락**: 지배적 변경 비용은 컴포넌트가 아니라 **prompt/tool-def/eval·모델 교체·신규 모델 온보딩** (렌즈1).
- 세분성 의존 — 컴포넌트 경계 정의 필요 (렌즈2).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "구성요소 변경 시 타 요소 영향 최소화."
- 제안: "컴포넌트·**prompt/tool-def·LLM 모델** 교체 시 타 요소 영향을 최소화하고(낮은 결합), 모델 교체에 무중단(model-agnostic)이다."

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | 평균 CIS ≤ 2개 컴포넌트 | **CIS p95 ≤ ◯** + 컴포넌트 경계 정의 명시 | tail 관리. ◯는 PoC-M1 |
| ② | — | **SDK 툴체인 변경 시 prompt/tool-def 수정 ≤ ◯** | agentic 1급 변경축 |
| ③ | — | **LLM 모델 교체가 워크플로우 무중단(model-agnostic)** | 모델 버전업 내성 |
| ④ | — | **신규 모델 온보딩 시간 ≤ ◯** | overview 140+ 모델 맥락 |

## 근거 (레퍼런스)
- **tail SLI(평균 금지)**: 변경 영향도 같은 분포는 평균이 tail을 숨김 → p95/max로 — SRE percentile 원칙. (§4 지연·유지보수 적용) — https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view
- **workflow-as-code 결합 분리·버저닝**: 노드 간 통신을 큐/계약(스키마)으로 분리하면 구현 교체가 인접 노드를 안 건드림. 워크플로우 버저닝(in-flight 구버전, 신규 신버전)·worker blue-green으로 무중단 배포, 모델 교체는 activity 내부 버전 교체로 흡수 — durable 엔진 versioning. (§4·렌즈3) — https://docs.temporal.io/temporal
- ⚠️ CIS ◯·온보딩 ◯시간은 우리 변경 시나리오로 확정.

## PoC 증명법

### PoC-M1: 변경 시나리오로 CIS p95 + 모델 교체 무중단을 측정한다
- **가설**: "대표 변경(컴포넌트·prompt·모델 교체) 시 CIS p95 ≤ ◯, 모델 교체가 워크플로우 무중단이다."
- **지표**: 변경 유형별 CIS 분포(p95/max), prompt/tool-def 수정 수, 모델 교체 시 무중단 배포 성공률, 신규 모델 온보딩 시간.
- **셋업**: mock 파이프라인을 큐/계약으로 분리 + 워크플로우 버저닝 + 변경 시나리오 세트(① 한 노드 구현 교체 ② SDK 컴파일러 플래그 변경 ③ LLM 모델 v→v+1 교체 ④ 신규 모델 온보딩).
- **절차**: ① 각 시나리오 적용 → ② 영향받은 컴포넌트/파일/계약 집계(CIS) → ③ 모델 교체 중 in-flight WF 무중단 확인 → ④ 온보딩 시간 측정.
- **합격(Exit)**: CIS p95 ≤ ◯, 모델 교체 무중단 성공률 100%, 온보딩 ≤ ◯.
- **규모/기간**: 변경 시나리오 4종 × 각 수 회, 약 1~2일.
- **리스크/한계(silent cap)**: CIS는 컴포넌트 경계 정의에 민감 — 경계 기준을 명시 log해야 비교 가능. mock의 결합도가 실제 시스템보다 낮으면 CIS 낙관 편향.

## DP·발표 영향
- **DP 연결**: DP-0004(A5/A8 Job/단계 단위 독립 배포)·DP-0005(캐시 계층)가 ①에, FR-0003(영향 범위 자동 분석)이 CIS 측정 데이터 제공 → 역검토. **DP-0004/0005가 prompt/모델 교체 내성을 명시 안 함** → workflow 버저닝·계약 분리 tactic 보강 권고.
- **severity Low** — 발표 우선순위는 낮으나, model-agnostic·온보딩 시간은 140+ 모델 운영 설득에 보조 지표로 유효.
