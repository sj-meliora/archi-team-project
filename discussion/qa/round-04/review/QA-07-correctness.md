# Review: QA-07 Correctness

> source: context/qa/QA-07-correctness.md + QAS-07-correctness.md
> verdict: **Sound ◎ / KPI ○** — ★★★ 만점 캡(92~98%) + judge κ 조건은 "100% = 난이도 부족" 통찰을 정식화한 모범. 잔여는 main 축(정답률)이 [발표 서사]라 실측 불가 + judge κ 0.8 단일 출처 · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(golden 정답률 90~98% 상한 캡 + κ 조건) 검증 무게중심.

## 원문 요약
- 정의: 자동 산출물이 정답·정책 부합. 일관성(QA-11)과 짝, QA-09/11 게이트 전제.
- 헤드라인: `golden 정답률 ≥90%`.
- ★ 급간: main = 단계별 golden 정답률 %(정방향, 100% 만점 아님). ★★★ 92~98%+κ≥0.8 / ★★☆ 90~92%+κ≥0.6 / ★☆☆ ≥90%+κ 검증완료 / 불합격 <90% 또는 100% 통과 또는 κ 미검증. 조건: LLM-as-judge κ≥0.8 선검증 + golden 난이도 적정.

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
**두 통찰이 모범**: (1) "100% 통과 = golden 난이도 부족 → 불합격"으로 상한 캡(MMLU 90%+ 변별력 상실 근거)과 (2) "judge가 틀리면 정답률 왜곡 → κ 선검증" 조건. 둘 다 eval 하네스의 핵심 함정을 정확히 자각. 다만 red-team 반박: **judge κ는 task별로 다르다** — frontier judge가 일반 QA에서 κ 0.81~0.87이라도, **NPU quantize config 정답성 판정** 같은 도메인 특화 task에선 judge 자신이 도메인 지식 부족으로 κ가 더 낮을 수 있다. 인용된 κ(o4-mini 0.873 등)는 일반 벤치라 우리 도메인과 apples-to-apples가 아니다. → round-05 Council이 도메인 특화 judge κ를 보강하거나 "κ는 우리 task로 재측정" 명시.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙4 적용 모범**: single-axis(정답률)지만 상한 캡 + κ 조건으로 "충분히 높되 만점 아님 + judge 신뢰"를 처리. 가짜 급간 아님. ◎.
- **급간 reasonableness**: ★☆☆ ≥90% · ★★☆ (90,92) · ★★★ [92,98]. **★★☆ 대역이 2%p로 매우 좁다**(90~92) — QA-04와 같은 "좁은 중급 대역" 문제. 정답률 90~92% 사이에 설계 대안이 몰리면 변별 안 됨. 또 ★★★ 상한 98%와 "100% 불합격" 사이 (98,100) 구간이 **무등급 공백** — 98.5% 정답률은 ★★★도 불합격도 아닌 미정의. → 권고 1.
- **하한 보정 정합(OI-9)**: §측정(43~45행 ≥90%+캡+κ), 등급표, 변경이력, counsel(상한 캡+조건), QAS-07(`≥90%`) 일치. 하한 90% 재배치 안 함(적정). glossary 없음. **OI-9 통과.** 단 QAS-07엔 캡·κ가 안 보임(table-only이라 정상이나, QAS Measure에 "★ 급간은 QA-07 등급 척도" 참조가 있는지 권고 2).
- **변별력 + main 축 실측불가(가장 날카로움)**: QA-11과 동형 — main 축(golden 정답률)이 **[발표 서사]**(golden set·judge 구축 [생략])라 PoC로도 실측 불가. ★ 급간이 모델 추정 별점에 그침. judge κ 조건도 "선검증"인데 golden 자체가 미구축이라 **순환**(정답률 측정 ← golden ← judge κ ← 인간 라벨 ← 미수행). High는 아니나 Med 사유.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
golden 정답률 = eval runner job(golden fan-out → judge 채점 → 인간 라벨 대조로 κ). judge 자체가 LLM이라 **judge의 비결정·비용**도 관리 대상(QA-11 pass^k와 동형). κ 검증은 인간 라벨 코퍼스 필요 → [생략]된 노동. runner KPI: `judge 채점 재현율(동일 산출물 2회 채점 일치)` + `golden 난이도 분포(정답률이 100%면 난이도 부족 자동 flag)`. eval 서브시스템이 QA-03/06 red-team 하네스와 공유 자산.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | 정확성 단일 관심사·일관성(QA-11) 직교 명문·ISO Functional Correctness 앵커 |
| Measurable | ○ | 정답률·rework·회귀 구체 + 상한 캡·κ 조건. 단 golden 미구축으로 실측 불가 |
| Realistic | ○ | 90% 진입선 현실, ★★★ 캡 정직. 단 ★★☆ 좁음 + (98,100) 공백 + κ 도메인 의존 |
| Consistent | ◎ | OI-9 통과, 일관성/정확성 직교 정합 |

**verdict: Sound ◎ / KPI ○ · Med** (round-02 ○/△ High → ★ 신설로 **KPI △→○ 개선**, 잔여 Med = main 축 실측불가·judge κ 순환·도메인 apples-to-apples).

## Stage 2 권고
1. **[★★☆ 대역 + 공백 · Med]** ★★☆ (90,92) 2%p 좁음 + (98,100) 무등급 공백 정리 — `제안: ★★★ 상한을 99%로 넓히되 100%만 불합격, ★★☆ [90,93)로 확장`.
2. **[judge κ 도메인 재측정 · Med]** 인용 κ(일반 벤치)는 우리 quantize 도메인과 다름 — `조건에 "κ는 우리 단계별 golden으로 재측정(일반 벤치 복제 아님)" 명시`. round-05 Council이 도메인 judge κ 보강.
3. **[main 축 실측불가 silent cap · Med]** golden 미구축으로 ★ 급간이 모델 추정임을 명시(QA-11과 동형, C3 군집). 산출 가능한 보조(rework율 등)를 보조 별점 축으로 검토.
4. **[QAS 참조 · Low]** QAS-07 Measure에 "★ 급간은 QA-07 등급 척도" 참조 추가(QA-06/11 QAS는 있음).
5. DP 역검토: eval/검증 서브시스템 신규 DP(related-dp 빈 상태, OI-7 1순위).
