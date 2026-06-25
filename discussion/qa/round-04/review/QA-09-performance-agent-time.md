# Review: QA-09 Performance — Agent 수행 시간 (per-node)

> source: context/qa/QA-09-performance-agent-time.md + QAS-09-performance-agent-time.md
> verdict: **Sound ◎ / KPI ○** — speedup 3배→1× 하한 보정은 협업형(METR 0.84×)·완전자율(InfEngine 8.6~22.7×) 대비를 잘 잡았으나, ★★★ 8배가 단일 출처(InfEngine)에 전적 의존 + 그 URL이 미래형 arXiv ID · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(speedup 3배→1×/3/8) 검증 무게중심.

## 원문 요약
- 정의: 1-pass 성공 노드 한정, 노드타입별 speedup, gaming 방지(커버리지).
- 헤드라인: `노드타입별 speedup >1배 (first-pass 성공분)`.
- ★ 급간: main = speedup 배수(정방향). ★★★ ≥8배 / ★★☆ 3~8배 / ★☆☆ 1~3배 / 불합격 ≤1배. 조건: first-pass 성공분만(QA-07 golden 게이트)·커버리지 ≥80%.

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
"속도만 재면 빠르게 틀리기가 최적해 → first-pass 성공분만 집계(QA-07 게이트)" + "easy-node cherry-picking → 커버리지 ≥80%"는 eval gaming 두 함정을 정확히 막은 모범. apples-to-apples 측면에서 **가장 정교한 대비**: 협업형(METR RCT 0.84×=오히려 감속, Copilot 2.26×) vs 완전자율 headless(InfEngine 8.6~22.7×)를 구분해 "우리는 headless 배치라 협업형보다 빠름"을 논거로 삼았다. 그러나 반박: (1) **★★★ 8배·★★☆ 상한이 InfEngine 단일 사례에 전적 의존** — InfEngine이 "assistant-type 8.6~22.7×"라는데 그게 우리 SDK 빌드 노드(IR/Quant/Compile)와 같은 작업 종류인지 불명. (2) 우리 노드의 상당 시간은 **외부 컴파일러 tool 실행**(우리 최적화 밖, silent cap)인데, speedup 분모(수동 baseline)에도 그 시간이 들어가면 배수가 희석/과장될 수 있다. → round-05 Council이 InfEngine 작업 종류·tool 시간 처리 보강.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙4 적용**: speedup을 main, first-pass·커버리지를 조건. 정확. ○.
- **급간 reasonableness**: ★☆☆ (1,3) · ★★☆ [3,8) · ★★★ ≥8. **비등간격(배수 스케일)** — speedup은 곱셈 척도라 1/3/8 비등간격이 자연스러움(자의적 등간격 아님). ○.
- **하한 보정의 모호함(가장 날카로움)**: 캘리브레이션 노트가 "하한 사실상 1×로 하향 보정"이라면서 §측정 헤드라인은 `>1배`, ★☆☆은 `1×~3배`. **그런데 변경이력·노트가 "3배는 ★★☆ 진입선으로 유지"라고도 함** — 즉 구 하한 3배가 (a) 폐기됐는지 (b) ★★☆로 올라갔는지 표현이 섞여 있다. 실질은 "합격 하한 = >1배, 3배 = ★★☆ 경계"로 정합하나, "하향 보정"과 "3배 유지"가 한 문단에 공존해 **독해 혼란**. → 권고 1.
- **하한 보정 정합(OI-9)**: §측정(35~37행 >1배, ★☆☆ 1~3배), 등급표, 변경이력, counsel(3배→1×), QAS-09(`1×~3배(3배=★★☆)`) **일치**. glossary 없음. **OI-9 통과**(표현 혼란은 정합성 결함은 아님).
- **변별력**: 구 3배 하한은 협업형 기준 비현실(보정 사유 정확). 신 >1배는 "사람보다 빠름"을 최소선으로 — ★☆☆ 비사문화. ★★★ 8배는 완전자율 사례 하단이라 비사문화. 양호.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
speedup = 수동 baseline 중앙값 / agent 중앙값. baseline 표본·숙련도 편차가 최대 난점(이미 silent cap). runner 오버헤드(큐+cold-start)/총시간 ≤15% 보조 KPI가 speedup의 "군더더기" 레버를 드러냄 — warm pool On/Off A/B. ★ 급간이 DP-0001(즉시 실행)·DP-0004(cold-start) 변별을 보상. runner KPI: `runner 오버헤드 비율이 speedup 배수와 분리 계측(외부 tool 시간 별도 버킷)` — tool 시간 희석을 막아야 ★ 신뢰.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | per-node 단일 관심사·Performance 3분할 명문·gaming 방지 |
| Measurable | ○ | speedup·first-pass·커버리지·오버헤드 구체 |
| Realistic | ○ | >1배 최소선 현실, ★★★ 8배 완전자율 대역. 단 InfEngine 단일 의존·tool 시간 희석 |
| Consistent | ◎ | OI-9 통과(표현 혼란은 정합 결함 아님) |

**verdict: Sound ◎ / KPI ○ · Med** (round-02 ○/△ Med → ★ 신설로 **KPI △→○ 개선**, 잔여 Med = ★★★ 단일출처+미래형 URL, first-pass 게이트가 QA-07 golden(미구축) 의존).

## Stage 2 권고
1. **[하한 표현 정리 · Low]** "3배→1× 하향 보정"과 "3배는 ★★☆ 유지"가 공존해 혼란 — `제안: "합격 하한 = >1배(★☆☆), 구 하한 3배는 ★★☆ 경계로 상향"으로 단일 표현 통일`.
2. **[★★★ 단일 출처 + URL 검증 → Council · Med]** ★★★ 8배가 InfEngine(arXiv 2602.18985, 미래형 ID) 단독 의존 — round-05 Council이 (a) ID 실재·정확 인용, (b) InfEngine 작업 종류가 우리 SDK 노드와 apples-to-apples인지, (c) 독립 출처 보강.
3. **[tool 시간 처리 · Low]** 외부 컴파일러 tool 실행 시간을 speedup 분모/분자에서 어떻게 처리하는지 명시(희석/과장 방지).
4. **[first-pass 게이트 의존 · Med]** speedup 집계가 QA-07 golden(미구축, [발표 서사])에 의존 — QA-07 닫힘과 묶임(C 교차 의존, OI-7/OI-8).
5. DP 역검토: DP-0001 1안이 latency만 보고 품질 게이트 무연결(OI-7).
