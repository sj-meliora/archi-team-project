---
id: QA-10
category: QA
importance: L
difficulty: M
source: pptx p.13
related-dp: [DP-0004, DP-0005]
related-fr: [FR-0003]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "평균 CIS(tail 은닉) → p95 + prompt/tool-def·모델 교체·온보딩 등 agentic 유지보수축 추가 (자세히 → ## 변경 이력)"
---

# QA-10 Maintainability — 모듈 교체 용이성

## 정의 / Refinement
컴포넌트·**prompt/tool-def·LLM 모델** 교체 시 타 요소로의 영향을 최소화하고(낮은 결합), **모델 교체에 무중단(model-agnostic)**이다. agentic 시스템 유지보수의 지배적 비용은 컴포넌트 자체가 아니라 **prompt·tool 정의·모델 교체·신규 모델 온보딩**이므로, 이 축을 KPI에 포함한다.

설계할 때 잡아야 할 두 가지 관점:

- **평균이 아니라 tail(p95)로 본다** — `평균 CIS ≤ 2`는 한 변경이 12개 컴포넌트를 건드려도 평균에 묻혀 tail 위험을 숨긴다. **p95(또는 max)**로 관리하고, CIS는 "컴포넌트"를 어떻게 세느냐에 민감하므로 **컴포넌트 경계 정의를 동반**해야 비교 가능하다. *CIS = Change Impact Scope(한 변경이 영향을 주는 컴포넌트 수).*
- **agentic 유지보수의 진짜 비용축** — SDK 컴파일러 플래그·IR 포맷이 바뀌면 agent의 prompt·tool 정의를 얼마나 고쳐야 하나(가장 잦은 변경). LLM 버전 업그레이드가 워크플로우를 깨면 안 된다(**model-agnostic 무중단**). overview의 **140+ 모델** 맥락에서 **신규 모델 온보딩 시간**도 1급 지표다.

> 이 QA는 "**변경의 국소성(낮은 결합)**"을 다룬다 — workflow-as-code 관점에서 "모듈 교체"는 activity 교체로 환원되고, 노드 간 통신을 큐/계약(스키마)으로 분리하면 한 노드 구현 교체가 인접 노드를 안 건드린다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `CIS p95` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **CIS p95 ≤ 3개 컴포넌트** `[주 KPI · PoC 대상]` + 컴포넌트 경계 정의 명시 — tail 관리
  - 쉽게: 한 컴포넌트를 바꿀 때 덩달아 손봐야 하는 컴포넌트 수가, (영향이 큰 상위 5%를 봐도) 3개 이하여야 한다. 평균이 아니라 p95로 봐야 "가끔 12개를 건드리는" 위험이 안 묻힌다. *p95 = 상위 5%를 뺀 95% 기준값.*
- **SDK 툴체인 변경 시 prompt/tool-def 수정 ≤ 2** — agentic 1급 변경축
  - 쉽게: SDK 컴파일러 플래그·IR 포맷이 바뀌어도, agent의 프롬프트·도구 정의는 2곳 이하만 고치면 되게 한다.
- **LLM 모델 교체가 워크플로우 무중단(model-agnostic)** — 모델 v→v+1 교체 시 in-flight WF 무중단
  - 쉽게: LLM 모델을 새 버전으로 갈아도 돌아가던 워크플로우가 멈추지 않아야 한다. 모델에 종속되지 않는 구조. *model-agnostic = 특정 모델에 묶이지 않음.*
- **신규 모델 온보딩 시간 ≤ 1일** — overview 140+ 모델 맥락
  - 쉽게: 새 NPU 타깃 모델 1건을 파이프라인에 태우는 준비가 하루 안에 끝나야 한다.

> 위 수치(p95 3개·수정 2·1일)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 변경 시나리오 측정으로 확정한다.
> 폐기: 旧 `평균 CIS ≤ 2개`(평균 단독) — tail 위험 은닉(한 변경이 12개를 건드려도 평균에 묻힘) + 컴포넌트 경계 정의 부재. → p95 + 경계 정의 + agentic 변경축(prompt/모델/온보딩)으로 확장.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 SRE tail 관리·workflow-as-code 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **tail SLI (평균 금지)** | 변경 영향도 분포는 평균이 tail을 숨김 → p95/max로 관리 | [p50/p95/p99 해설](https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view) |
| **workflow-as-code 결합 분리·버저닝** | 노드 간 통신을 큐/계약(스키마)으로 분리 → 구현 교체가 인접 노드 미간섭. 워크플로우 버저닝(in-flight 구버전, 신규 신버전)·worker blue-green으로 무중단 배포, 모델 교체는 activity 내부 버전 교체로 흡수 | [Temporal — 문서(versioning)](https://docs.temporal.io/temporal) |

> ⚠️ 레퍼런스의 패턴은 **정당화용**이며 수치를 복제하지 않는다. CIS p95(예시 3개)·온보딩(예시 1일)은 위 [검증 전략](#검증-전략)의 변경 시나리오로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 변경 시나리오 모델**로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **CIS p95** `[주]` | **DP-0004 A5/A8**(Job/단계 단위 독립 배포 [Maintainability]★★☆) · **FR-0003**(영향 범위 자동 분석이 CIS 측정 데이터 제공) | **▶ 실제 제작:** 변경 시나리오 모델 — mock 파이프라인을 큐/계약으로 분리 + 워크플로우 버저닝 + 변경 시나리오 세트(① 한 노드 구현 교체 ② SDK 플래그 변경 ③ 모델 v→v+1 ④ 신규 모델 온보딩) → 각 시나리오의 **영향받은 컴포넌트/파일/계약 집계(CIS 분포 p95/max)** |
| prompt/tool-def 수정 ≤ 2 | 큐/계약 분리(prompt·tool 정의를 컴포넌트 경계 안에 격리) | 보조 모델: SDK 플래그 변경 시나리오에서 수정 파일/정의 수 집계 |
| 모델 교체 무중단 | 워크플로우 버저닝(in-flight 구버전, 신규 신버전) + worker blue-green | 보조 모델: 모델 교체 중 in-flight WF 무중단 배포 성공률 확인 |
| 신규 모델 온보딩 ≤ 1일 | activity 내부 버전 교체(model-agnostic) | 보조 모델: 온보딩 시나리오 소요 단계 수·시간 측정 |

> 가정·한계: **CIS는 컴포넌트 경계 정의에 민감** — 경계 기준을 명시해야 비교 가능. mock의 결합도가 실제 시스템보다 낮으면 CIS가 낙관 편향. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — silent cap으로 명시.

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-10-maintainability.md) (red team verdict: **Sound ○ / KPI ○ — Low** — 건강, 평균→tail 전환 + agentic 축 추가)

**무엇이 문제였나 (review 지적)**
- **건강한 QA** — 모듈성(낮은 결합) 단일 관심사 명확, CIS 측정 가능.
- **평균의 함정**: `평균 CIS ≤ 2`는 tail 은닉(한 변경이 12개 건드려도 평균에 묻힘) → p95/max.
- **agentic 유지보수축 누락**: 지배적 변경 비용은 컴포넌트가 아니라 **prompt/tool-def·모델 교체·신규 모델 온보딩**. 세분성 의존(컴포넌트 경계 정의 필요).

**무엇을 바꿨나 (반영)**
- **정의**: "컴포넌트·prompt/tool-def·LLM 모델 교체 내성 + model-agnostic 무중단"으로 확장. workflow-as-code(activity 교체·큐/계약 분리) altitude 명시.
- **KPI 확장**: 旧 `평균 CIS ≤2` → ① `CIS p95 ≤3 + 컴포넌트 경계 정의` ② `prompt/tool-def 수정 ≤2` ③ `모델 교체 무중단(model-agnostic)` ④ `신규 모델 온보딩 ≤1일`.
- 짝 시나리오 `QAS-10`의 자극(prompt/모델 교체 추가)·Response·Measure를 동기화.

**남은 일 (이 라운드에서 미반영)**
- **DP-0004/0005가 prompt/모델 교체 내성을 명시 안 함** → workflow 버저닝·계약 분리 tactic 보강 역검토 (`open-issues.md` 트래킹 대상). FR-0003(영향 범위 자동 분석)이 CIS 측정 데이터를 제공하는지 역검토.
- severity Low — 발표 우선순위는 낮으나 model-agnostic·온보딩 시간은 140+ 모델 운영 설득의 보조 지표로 보존.
- CIS `p95 3개`·온보딩 `1일`은 **예시값**이며 변경 시나리오 측정으로 확정.
