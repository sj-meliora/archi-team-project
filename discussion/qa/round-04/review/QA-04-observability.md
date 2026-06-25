# Review: QA-04 Observability

> source: context/qa/QA-04-observability.md + QAS-04-observability.md
> verdict: **Sound ◎ / KPI ○** — ★★★ 100% 미만(≥98%) 상한 캡은 CoT 비공개를 정직하게 반영한 모범. 잔여는 ★★☆ 대역(95~98%)이 매우 좁아 변별력 빈약 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(trace 완전성 ≥95%, 상한 캡 ≥98%) 검증 무게중심.

## 원문 요약
- 정의: agent loop 전구간 span 캡처 + traces/metrics/alerting 3축. 다수 QA 측정 인프라 토대.
- 헤드라인: `trace 완전성: 안전 100% / 일반 ≥95%`.
- ★ 급간: main = 일반 액션 완전성 %(정방향). ★★★ ≥98%(100% 미만 캡) / ★★☆ 95~98% / ★☆☆ ≥95%. 게이트: 안전 액션 100%·event-history 100%.

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
**★★★ 상한 캡(100% 미만)이 agentic 통찰의 정수다.** "LLM 내부 CoT는 비공개라 span 완전성 ≠ 의미 재현율" → 100% trace를 단정하면 과대주장. 이를 캡으로 못박은 건 세트에서 가장 정직한 처리. OTel GenAI semconv(invoke_agent/chat/execute_tool span)는 실재하는 표준이고 우리 측정 대상(agent 단계 계측)과 apples-to-apples로 잘 맞는다 — C1 우려가 가장 낮은 케이스. 단 보강: "완전성 %"가 **span 존재율**이지 **각 span의 내용 충실도**(prompt/context가 잘렸는지)는 안 잰다 — 존재하지만 truncate된 span도 100% 카운트될 위험. round-05 Council이 "완전성 = 존재 AND 비-truncation" 정의를 보강할 것.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙4·5 적용 정확**: 안전 액션 100%·event-history 100%를 게이트로, gradable한 일반 완전성 %에 별점. ◎.
- **급간 reasonableness**: ★☆☆ ≥95% · ★★☆ (95,98) · ★★★ [98,100). **★★☆ 대역이 3%p로 매우 좁다** — auto-instrumentation이 표준 span을 거의 다 잡으면 95~98% 사이에 설계 대안이 몰려 변별이 안 될 수 있다. 또 ★☆☆(≥95%)과 ★★☆(>95%)의 경계가 정확히 95%에서 갈려 **≥95 vs >95 표기 모호**(95.0%는 ★☆☆인가 ★★☆인가). → 권고 1.
- **하한 보정 정합(OI-9)**: §측정(32~34행 ≥95% 유지+★★★ 캡), 등급표, 변경이력, counsel(상한 캡), QAS-04(`안전 100%·일반 ≥95%`) 일치. **하한은 재배치 안 함(95% 적정)** — 보정은 상한 캡만. glossary 없음. **OI-9 통과.**
- **변별력**: 하한 95%가 비현실적으로 높지 않아 ★☆☆ 비사문화. ★★★ 캡(≥98%)으로 100% 사문화도 회피. 단 ★★☆ 좁은 대역이 변별력 약점.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
trace 완전성 = OTel span 자동계측 커버리지. event-history 100% 게이트는 durable 엔진의 결정론적 replay라 **구조적으로 보장**(가정 아님) — 게이트로 둔 것 정확. 일반 액션 완전성은 커스텀 tool·외부 호출 span 누락이 변수 → ★★☆/★★★(98%) 변별엔 N개 액션 종류 분모가 충분해야. runner KPI: `span 누락 카테고리별 분해(표준 vs 커스텀 tool vs 외부 호출)` — 어디서 2%가 빠지는지 보여야 ★★★ 도달 가능 여부 판정.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | 관측 인프라 단일 관심사·다QA 공급 관계 명문 |
| Measurable | ○ | span 완전성·event-history·MTTD 구체. "존재율 vs 충실도" 구분 보강 여지 |
| Realistic | ◎ | ★★★ 캡이 CoT 비공개 정직 반영, 하한 95% 적정 |
| Consistent | ◎ | OI-9 통과, event-history 게이트 정합 |

**verdict: Sound ◎ / KPI ○ · Low** (round-02 ○/○ Low 유지, ★ 캡 신설로 정직성 보강).

## Stage 2 권고
1. **[경계 표기 명확화 · Low]** ★☆☆ ≥95%와 ★★☆ >95% 경계 모호 — `제안: ★☆☆ [95,97), ★★☆ [97,98), ★★★ [98,100) 식으로 ★★☆ 대역 명시 또는 ≥/> 일관`.
2. **[완전성 정의 보강 · Low]** "완전성 = span 존재 AND 비-truncation(prompt/context 잘림 없음)"으로 round-05 Council이 정의 강화 — 존재하나 잘린 span 과대 카운트 방지.
3. **[누락 카테고리 분해 · Low]** ★★★ 98% 도달 판정엔 span 누락 카테고리별 분해 필요(렌즈3).
4. DP 역검토: DP-0003이 span 수준 trace를 보장하는지(규칙 기반 한정 시 어디까지)(OI-7).
