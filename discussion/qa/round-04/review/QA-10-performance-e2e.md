# Review: QA-10 Performance — E2E 개발 시간

> source: context/qa/QA-10-performance-e2e.md + QAS-10-performance-e2e.md
> verdict: **Sound ◎ / KPI ○** — E2E 6h→24h 재배치는 "보수적 하한(★☆☆ 사문화)" 반대 방향 문제를 정확히 교정한 세트 유일 케이스. 잔여는 ★★★ 2h margin 근거 약함 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(E2E p95 6h→24h 재배치) 검증 무게중심.

## 원문 요약
- 정의: 모델 1건 E2E 응답·처리량. handoff(20GB 전달)는 하위 요소.
- 헤드라인: `E2E latency/모델 ≤24h (p95)`(보정 전 ≤6h).
- ★ 급간: main = E2E p95(역방향). ★★★ ≤2h / ★★☆ 2~6h(p50 ≤6h) / ★☆☆ 6~24h / 불합격 >24h. 조건: throughput ≥50/일·전달 무결성 100%.

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
"이름이 약속한 E2E를 실제로 측정"(전달 5% 단독 → E2E latency top-line)은 정의↔KPI mislabel을 복구한 round-01 산물. ★ 급간 근거(배치 ML 파이프라인 SLA 6~24h)는 우리 워크로드(배치 SDK 빌드)와 **도메인 정합이 좋은 편** — apples-to-apples 우려 낮음. 다만 agentic 고유 변수: E2E엔 **agent 루프(LLM 호출 대기·rate-limit)**가 포함되는데, ★ 근거(배치 ML SLA)는 전통 ML 파이프라인이라 **LLM 호출 지연·rate-limit 큐잉**을 반영 안 한다 — 외부 LLM이 느리면 E2E가 배치 ML 관례를 넘을 수 있다. → round-05 Council이 "agentic E2E = 배치 ML + LLM 큐잉" 가산 명시.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙2 반대 방향 교정(세트 유일)**: 다른 보정은 "비현실 하한 → 완화"인데, QA-10은 **"보수적 하한(≤6h)이 ★☆☆을 사문화"** → 하한을 24h로 **넓혀** ★☆☆을 살린 유일 케이스. round-04 특별 초점이 의심한 "★☆☆ 사문화 군집"의 정확한 사례이자 교정. ◎.
- **급간 reasonableness**: ★★★ ≤2h · ★★☆ 2~6h · ★☆☆ 6~24h. **비등간격(2/4/18h, 로그 가까움)** — 배치 ML 6~24h 대역을 ★☆☆로, 그 1/3(6h)을 ★★☆, 1/3(2h)을 ★★★로. 합리적.
- **★★★ 2h margin 약함**: "배치 표준 하한 6h의 1/3 = 2h, 이론적으로 더 낮출 수 있으나 Quantize/Compile compute가 critical path라 PoC margin으로 2h" → **왜 1/3(2h)인지** 정량 근거 없음(C2 군집). 1h도 3h도 아닌 이유 부재.
- **하한 보정 정합(OI-9)**: §측정(36~38행 ≤24h), 등급표, 변경이력, counsel(6h→24h), QAS-10(`≤24h; 6h=★★☆`) **일치**. glossary 없음. **OI-9 통과.**
- **변별력**: 구 ≤6h를 합격 하한으로 쓰면 ★☆☆ 사문화(보정 사유 정확). 신 24h(배치 nightly 관례)로 ★☆☆ 부활. 24h 초과 = "수동 수일과 차별성 약화"로 불합격 — 진입선이 질적으로 타당. ◎.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
E2E = compute + agent 루프 + 큐 대기 + handoff. claim-check(원격, A5) vs 로컬(A8) 2모드 부하시험으로 단계 분해. **조건 C(throughput ≥50/일 부하 하 측정)가 핵심** — latency는 부하 낮으면 자명히 낮아지므로 throughput SLO 충족 부하에서만 측정해야 ★ 유효(QA-01 공유 축). 정확한 분리. runner KPI: `E2E p95를 단계(compute/agent/큐/handoff)별 분해 + critical path 식별` — mock compute가 실제 Quantize/Compile 분포 미근사 시 왜곡(silent cap).

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | E2E 단일 관심사·Performance 3분할 명문·정의↔KPI mislabel 복구 |
| Measurable | ○ | E2E p50/p95·throughput·전달 무결성 구체 |
| Realistic | ◎ | 24h가 배치 ML 관례 정합, ★☆☆ 사문화 교정. ★★★ 2h margin만 약함 |
| Consistent | ◎ | OI-9 통과, QA-01 throughput 정렬 |

**verdict: Sound ◎ / KPI ○ · Low** (round-02 ○/○ Low 유지, ★☆☆ 사문화 교정으로 변별력 개선).

## Stage 2 권고
1. **[★★★ 2h margin 근거 · Low]** "6h의 1/3 = 2h"의 정량 근거 보강 — `제안: "완전자율 병렬 파이프라인 도달 대역 + Quantize/Compile critical path 흡수 ≈2h"로 명시`.
2. **[agentic E2E 가산 · Low]** 배치 ML SLA(6~24h)에 LLM 호출 큐잉·rate-limit 지연이 추가될 수 있음을 ## 등급 척도에 명시(전통 ML과 차이).
3. **[근거 URL 검증 → Council · Low]** 배치 ML SLA 출처(domo·datadef)를 round-05 Council이 실재 확인.
4. DP 역검토: DP-0004 결정 산식(5% budget → A5 vs A8)은 실측·노드 사양 의존(OI-7).
