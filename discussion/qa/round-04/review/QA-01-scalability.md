# Review: QA-01 Scalability

> source: context/qa/QA-01-scalability.md + QAS-01-scalability.md
> verdict: **Sound ◎ / KPI ○** — efficiency 0.8→0.70 재배치는 USL 근거에 충실하나 **단일 출처(WSO2 0.72)에 경계가 매달리고 margin이 0.02~0.05로 흔들림** · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(scaling efficiency 하한 0.8→0.70) 검증 무게중심.

## 원문 요약
- 정의: 부하↑ 시 처리량 비례 유지, 1차 병목 = 외부 LLM rate-limit, backlog 오토스케일.
- 헤드라인 KPI: `scaling efficiency ≥0.70` (보정 전 0.8).
- ★ 급간: ★★★ ≥0.85 / ★★☆ 0.75~0.85 / ★☆☆ 0.70~0.75 / 불합격 <0.70. 조건: rate-limit 헤드룸 ≥20% 고정(2-index → main+조건).

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
★ 경계가 LLM rate-limited 워크로드를 직접 잰 USL 회귀가 아니라 **일반 HPC USL(SPARCcenter SPEC SDM91, 1990년대 CPU 벤치)**에서 빌려왔다. agentic 시스템의 진짜 확장 한계는 coherency(β)·contention(α)이 아니라 **외부 LLM 계정 전역 TPM/RPM**인데, 이걸 `측정 조건: 헤드룸 ≥20% 고정`으로 **밖으로 빼버려서** ★ 급간이 잰 efficiency는 "rate-limit이 포화되지 않은 가상 조건의 순수 큐잉 효율"이다 — 즉 ★★★(0.85)를 받아도 **실운영에서 계정 한도에 막히면 그 efficiency는 의미 없다.** apples-to-apples 측면: SPARCcenter CPU 확장과 LLM-bound worker 확장은 병목 구조가 달라 0.72라는 숫자가 우리 천장을 대표한다는 보장이 없다. → round-05 Council이 LLM-bound 워크로드의 USL 회귀 사례를 보강할 것.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙4 분해 타당**: efficiency를 main, 헤드룸을 조건으로 분리한 것은 정확. rate-limit 포화로 등급이 깎이는 걸 배제한 의도 명확. ○.
- **급간 reasonableness**: ★☆☆ [0.70,0.75) · ★★☆ [0.75,0.85) · ★★★ [0.85,∞). **폭이 0.05/0.10/∞로 비등간격** — 0.75(HPC "Good")와 0.85(이론 0.90−margin)를 데이터포인트로 잡은 합리적 배치. 등간격 자의성 아님.
- **margin 자의성(가장 날카로움)**: ★★★ 0.85는 "이론 0.90 − margin 0.05", ★☆☆ 하한 0.70은 "필드 0.72 − margin 0.02". 캘리브레이션 노트가 **"margin ≈0.02~0.05"로 범위를 명시**해 자인하나, 이는 **경계마다 다른 margin을 쓴다는 자백**이다(QA-06 C2와 동형). 0.72에서 0.70으로 0.02만 뺀 건 근거가 약하다 — 왜 0.05가 아닌가? → C2 군집.
- **하한 보정 정합(OI-9)**: §측정(35행 ≥0.70), 등급표, 변경이력, counsel(0.8→0.70), QAS-01(`≥0.70 ★☆☆`) **5곳 일치**. glossary 수치 없음. **OI-9 통과.**
- **변별력**: 구 0.8은 "필드 우수도 0.72"라 ★★★ 사문화였음(보정 사유 정확). 신 0.70은 0.72 바로 아래라 **★☆☆이 거의 사문화 위험** — 합격 진입선(0.70)과 필드 우수(0.72)가 0.02밖에 안 떨어져 [0.70,0.75) ★☆☆ 대역이 매우 좁다. 보정이 살짝 과해 ★☆☆ 변별력이 빈약. Low 잔여.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
efficiency = (부하 2배 시 처리량 배수 ÷ 2)는 backlog 기반 오토스케일·공유 풀 시뮬로 측정. PoC가 가정 파라미터(서비스시간·도착률·cold-start) 기반이라 efficiency 절대값이 가정에 좌우 → ★ 경계 신뢰엔 **민감도 분석**이 필요. runner KPI: `efficiency 추정이 cold-start·라우팅 지연 파라미터에 ±0.05 이내로 robust` 미달 시 ★ 경계 신뢰 불가 silent cap.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | throughput 단일 관심사·QA-09/10 경계 명문 |
| Measurable | ○ | efficiency·헤드룸 임계 구체. PoC 가정 민감 |
| Realistic | ○ | 0.70이 필드 0.72 정합. 단 ★☆☆ 대역(0.70~0.75)이 좁아 변별력 빈약 |
| Consistent | ◎ | 5곳 0.70 일치(OI-9 통과) |

**verdict: Sound ◎ / KPI ○ · Low** (round-02 ○/○ Low 유지, ★ 신설로 변동 없음).

## Stage 2 권고
1. **[margin 규칙화 · Low]** ★☆☆ 0.70(필드 0.72−0.02)과 ★★★ 0.85(이론 0.90−0.05)의 margin 불일치를 규칙화 — `제안: 둘 다 동일 margin 규칙 적용 또는 "★☆☆은 필드 우수 바로 아래라 margin 최소"임을 명시`.
2. **[★☆☆ 대역 폭 · Low]** [0.70,0.75) 0.05 폭이 ★☆☆ 변별엔 좁음 — 하한을 0.68 정도로 더 내려 ★☆☆ 대역을 넓힐지 검토(또는 좁은 채로 두되 silent cap).
3. **[LLM-bound USL 근거 → Council · Low]** SPARCcenter CPU USL이 아닌 LLM-bound worker 확장의 efficiency 사례 보강(round-05).
4. **[민감도 silent cap · Low]** efficiency 추정이 가정 파라미터에 robust한지 ## 검증 전략에 명시.
5. DP 역검토: DP-0001/0004의 rate-limit headroom·admission control·bounded queue 미명시(OI-7).
