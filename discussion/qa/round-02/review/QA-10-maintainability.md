# Review: QA-10 Maintainability — 모듈 교체 용이성 (round-02 재평가)

> source: `context/qa/QA-10-maintainability.md` + `QAS-10-maintainability.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound ○ / KPI ○ — Low**
> verdict(round-02): **Sound ○ / KPI ○ — Low** — 평균 CIS(tail 은닉) → p95 + prompt/모델 교체·온보딩 축 추가로 건강 유지·강화. 잔존은 DP-0004/0005 교체내성 역검토(OI-7)·컴포넌트 경계 정의 의존뿐 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-10 = **[반영]** (평균CIS → CIS p95 + prompt/모델 교체·온보딩 축). 이월: DP-0004/0005 교체내성 역검토(OI-7).

## 원문 요약 (반영 후)
- **정의**: 컴포넌트·prompt/tool-def·LLM 모델 교체 시 영향 최소화 + 모델 교체 무중단(model-agnostic). agentic 유지보수의 지배 비용 = prompt·tool 정의·모델 교체·신규 모델 온보딩.
- **KPI**: 주 = `CIS p95 ≤3 + 컴포넌트 경계 정의`. 보조 = `SDK 변경 시 prompt/tool-def 수정 ≤2` · `모델 교체 무중단(model-agnostic)` · `신규 모델 온보딩 ≤1일`.
- **QAS-10**: 자극(prompt/모델 교체)·Measure 동기화.

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01 핵심 지적("agentic 유지보수의 진짜 비용은 컴포넌트가 아니라 prompt·tool 정의·모델 교체·신규 모델 온보딩")이 정확히 반영됐다. 일반 분산시스템 어휘(CIS)에 머물던 QA가 agentic 변경축을 1급 KPI로 올렸다:
- `SDK 변경 시 prompt/tool-def 수정 ≤2` — 가장 잦은 변경(컴파일러 플래그·IR 포맷 변경 시 prompt 영향).
- `모델 교체 무중단(model-agnostic)` — LLM v→v+1이 워크플로우를 깨면 안 됨. overview 140+ 모델 맥락의 `온보딩 ≤1일`도 1급 지표로. **닫혔다.**

- **잔여(silent cap)**: `모델 교체 무중단`이 *기능적 무중단*(WF 안 멈춤)인지 *행동적 무중단*(같은 결정 유지)인지 모호 — 모델을 갈면 WF는 안 멈춰도 **결정 분포가 달라질 수 있다**(QA-09 일관성·NQA-B 정확성 영향). "무중단"이 in-flight 연속성만 보장하고 결정 동등성은 QA-09/NQA-B 소관임을 cross-link 권고(Low). QA-02 폴백 일관성과 동일 뿌리.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **평균의 함정 해소 확인**: round-01의 `평균 CIS ≤2`(한 변경이 12개를 건드려도 평균에 묻혀 tail 은닉)가 **`CIS p95 ≤3 + 컴포넌트 경계 정의`**로 교정. tail SLI로 전환되고, CIS가 "컴포넌트를 어떻게 세느냐"에 민감하다는 점을 **컴포넌트 경계 정의 동반**으로 잡았다. **measurable 강화.**
- **Sound ○ 유지**: 변경의 국소성(낮은 결합) 단일 관심사. workflow-as-code("모듈 교체"=activity 교체) altitude 명시로 경계 명확. consistency ○(QAS-10 동기화).
- **남은 형식 결함(경미)**: `CIS는 컴포넌트 경계 정의에 민감 + mock 결합도가 실제보다 낮으면 낙관 편향`이 silent cap. p95 3개 합격이 경계 정의·mock 충실도에 좌우 → 경계 기준을 KPI 옆에 *고정*함을 명문화(이미 "경계 정의 명시"로 부분 반영). Low.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **workflow-as-code 결합 분리·버저닝**(round-01 렌즈3) → 정의·검증 전략에 "노드 간 통신을 큐/계약(스키마)으로 분리 → 구현 교체가 인접 노드 미간섭", "워크플로우 버저닝(in-flight 구버전·신규 신버전) + worker blue-green 무중단 배포", "모델 교체는 activity 내부 버전 교체로 흡수"가 정확히 환원됐다. **양호.**
- **잔여(DP 위임, OI-7)**: KPI를 **DP-0004 A5/A8 + FR-0003**에 귀속시키나, OI-7이 명시하듯 **DP-0004/0005가 prompt/모델 교체 내성(workflow 버저닝·계약 분리)을 명시 안 함**. FR-0003(영향 범위 자동 분석)이 CIS 측정 데이터를 공급하는지도 역검토 필요 → DP 디스커션 위임.
- **runner 측 KPI**(재확인): 워크플로우 버전 공존 수(blue-green 중 in-flight 구버전·신규 신버전 동시 가동), 무중단 배포 성공률, 계약(스키마) 변경 시 영향 노드 수, activity 교체 시 인접 노드 재배포 0.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | ○ | **○** | 변경 국소성 단일 관심사 + workflow-as-code altitude 명시 |
| KPI가 측정 가능한가 | ○ | **○** | 평균→p95 tail SLI 전환 + 컴포넌트 경계 정의 동반. agentic 변경축 추가 |
| KPI가 현실적/적절한가 | ○ | **○** | model-agnostic·온보딩 현실적. 단 "무중단" 의미(기능 vs 행동)·경계 민감도는 Low 보강 |
| 정의↔KPI↔QAS 일치 | ○ | **○** | QAS-10 자극(prompt/모델 교체)·Measure 동기화 확인 |

**verdict 변화: Sound ○→○ / KPI ○→○ · severity Low→Low.** round-01 Low의 권고(평균→tail·agentic 축)는 반영돼 건강 유지·강화. 잔여는 DP 역검토뿐.

## Stage 2 권고 (round-02)

건강한 QA라 **DP 위임·cross-link만** 남긴다:

- **(DP 디스커션 위임, OI-7)** DP-0004/0005에 **prompt/모델 교체 내성**(workflow 버저닝·계약 분리·blue-green) 명시. FR-0003이 CIS 측정 데이터 공급하는지 역검토.
- **(무중단 의미 명시, Low)** `모델 교체 무중단`이 **기능적 무중단(in-flight 연속성)**임을 명시 + 결정 동등성은 QA-09/NQA-B 소관임을 cross-link.
- **(경계 고정, Low)** CIS 컴포넌트 경계 기준을 KPI 옆에 고정 명시(낙관 편향·비교가능성).
- **(예시값 확정)** `p95 3개·수정 2·1일`은 변경 시나리오 측정으로 확정.
