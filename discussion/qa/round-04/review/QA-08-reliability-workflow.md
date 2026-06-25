# Review: QA-08 Reliability — Workflow 간 독립성

> source: context/qa/QA-08-reliability-workflow.md + QAS-08-reliability-workflow.md
> verdict: **Sound ◎ / KPI ○** — latency 증가 10→25% 재배치는 무격리 바닥(+100%·56%)과 간격 확보가 합리적. 잔여는 ★★★ 5% margin 근거 약함 + 보조 근거 URL이 미래/가공 의심 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(타 WF latency 증가 10→25%) 검증 무게중심.

## 원문 요약
- 정의: 한 WF 장애·자원 폭주가 타 WF에 무영향(blast-radius 봉쇄). 진짜 공유 도메인 = LLM rate-limit 풀·공유 캐시.
- 헤드라인: `타 WF 중단 ≤1% AND latency 증가 ≤25%`.
- ★ 급간: main = 폭주 주입 하 타 WF latency(p95) 증가율(역방향). ★★★ ≤5% / ★★☆ ≤10% / ★☆☆ ≤25% / 불합격 >25% 또는 중단율>1% 또는 쿼터침범>0. 조건: 중단율 ≤1%·쿼터침범 0.

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
"noisy-neighbor 실체 = 쿼터 독점(한 WF 폭주가 토큰 예산·rate-limit 빨아들임)"을 정확히 짚고 WF별 token-bucket을 격리에 포함한 것은 agentic 통찰. 외부 LLM rate-limit이 계정 전역이라 token-bucket은 client-side 분배만 보장(전역 풀 마르면 전 WF 동반 저하)이라는 silent cap도 정직. 그러나 ★ main 축(latency 증가율)은 **compute/큐 간섭**을 재는 전통 격리 지표인데, agentic의 진짜 공유 장애는 **쿼터**다 — 쿼터 침범은 0건 게이트로만 다루고 ★ 변별엔 안 들어간다. 즉 ★★★(latency ≤5%)를 받아도 전역 쿼터가 마르면 무의미. apples-to-apples: 인용한 "p99 doubling·CPU 56% degradation"은 **스토리지/CPU noisy-neighbor**(simplyblock)이지 LLM 토큰 쿼torque 간섭이 아니다.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙4·5 적용**: latency를 main, 중단율을 조건, 쿼터침범 0을 게이트로. 정확. ○.
- **급간 reasonableness**: ★★★ ≤5% · ★★☆ ≤10% · ★☆☆ ≤25%. **비등간격(5/5/15)** — 무격리 바닥(+100%)과 충분한 간격 확보 의도. ★☆☆ 폭(10~25%, 15%p)이 넓어 변별 여유. 합리적 배치.
- **margin 근거 약함**: ★★★ 5%는 "무간섭 이상에 PoC margin 두고 5%"인데 **왜 5%인지 근거 없음**(0%도 3%도 아닌). ★☆☆ 25%는 "무격리 +100% 바닥의 한참 위" → 정성적. margin 정량 근거 부재(C2 군집).
- **하한 보정 정합(OI-9)**: §측정(38~40행 ≤25%), 등급표, 변경이력, counsel(10→25%), QAS-08(`≤25% ★☆☆; 10%=★★☆`) **일치**. glossary 없음. **OI-9 통과.**
- **변별력**: 구 10% 하한은 건강했으나 ★1~3 급간이 안 떴음 → 10%를 ★★☆로 올리고 25%를 합격 하한으로(★ 급간 확보). 보정이 **변별력을 만들기 위한 것**이지 물러지게 한 게 아님 — 단 25%까지 합격을 넓힌 게 "격리 빡빡함"을 약화시키진 않는지(원래 10%가 건강했는데). Low 잔여.
- **출처 URL 의심**: 근거 `arXiv 2604.03145`·`arXiv 2602.18985`(QA-09)·`arXiv 2511.07413`(QA-07) 등은 **2026년 이후 arXiv ID**(2604=2026년 4월)로 보여 — 본 리뷰 시점(2026-06) 기준 일부는 실재하나, **ID 형식이 미래/가공일 가능성**을 round-05 Council이 1차 출처로 반드시 검증해야(웹 도구 없어 확인 불가).

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
latency 증가 = bulkhead·큐별 동시성·token-bucket On/Off A/B. ★ 급간이 격리 메커니즘(전용 큐·rate limiter) 유무를 직접 보상 → 설계 대안 변별로 적합. 폭주 주입 강도·WF 수가 가정 파라미터라 ★ 경계 신뢰엔 민감도 필요. runner KPI: `폭주 강도 sweep에서 타 WF p95 증가가 격리 On에서 단조 평탄, Off에서 급증` + `쿼터 침범 0이 폭주 강도 무관 유지`.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | 격리 단일 관심사·QA-02(복구) 경계 양방향 명문·쿼터/상태/스케줄러 채널 명시 |
| Measurable | ○ | latency·중단율·쿼터·캐시오염 4축 구체 |
| Realistic | ○ | 25% 하한이 무격리 바닥과 간격 확보. ★★★ 5% margin 근거 약함 |
| Consistent | ◎ | OI-9 통과 |

**verdict: Sound ◎ / KPI ○ · Low** (round-02 ○/○ Low 유지, 세트 모범).

## Stage 2 권고
1. **[★★★ 5% margin 근거 · Low]** "무간섭 이상 + PoC margin 5%"의 5% 정량 근거 보강 — `제안: "격리 시 tail 안정화 관측 대역의 하단 + 폭주 강도 가정 흡수 ≈5%"로 명시`.
2. **[출처 URL 검증 → Council · Med]** arXiv 2604.03145(noisy-neighbor causal inference) 등 미래형 ID를 round-05 Council이 1차 출처로 실재·정확 인용 확인. **세트 전반 공통 점검**(C 추가).
3. **[쿼터를 ★에 · Low]** agentic 진짜 공유 장애(쿼터)가 ★ 변별엔 빠지고 게이트로만 — 쿼터 간섭을 보조 별점 축으로 검토.
4. DP 역검토: DP-0004/0005 격리 — 특히 DP-0005 2안 공유 캐시 채택 시 ④(오염 전파 0)가 R-1과 충돌(OI-7).
