# Review: QA-06 Reliability — Workflow 간 독립성 보장 (round-02 재평가)

> source: `context/qa/QA-06-reliability-workflow.md` + `QAS-06-reliability-workflow.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound ○ / KPI ○ — Med**
> verdict(round-02): **Sound ○ / KPI ○ — Low** — 건강 KPI 유지 + 자원-쿼터 격리·캐시오염 KPI 추가 + QA-02 경계 양방향 명문화로 C3 닫힘. 잔존은 DP-0004/0005 격리 역검토(OI-7)·외부 rate-limit 계정전역 한계뿐 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-06 = **[반영]** (건강 KPI 유지 + 쿼터 침범0·캐시오염0 + QA-02 경계 명문화). 이월: DP-0004/0005 격리 역검토(OI-7); 외부 rate-limit 계정전역 silent cap.

## 원문 요약 (반영 후)
- **정의**: 한 WF의 장애·자원 폭주가 타 WF의 실행·지연·토큰/rate-limit 쿼터·공유 상태에 무영향(blast-radius 봉쇄). 진짜 공유 장애 도메인 = LLM rate-limit 풀·공유 캐시. QA-02(복구)↔QA-06(격리) 경계 양방향 명문화.
- **KPI**: 주 = `타 WF 중단 ≤1% AND latency 증가 ≤10%`(건강 유지). 보조 = `쿼터 침범=0(token-bucket)` · `공유 캐시 오염 전파=0`.
- **QAS-06**: 자극에 자원 폭주 추가, Measure 동기화.

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01의 핵심 지적("agentic 진짜 공유 장애 도메인은 노드가 아니라 LLM rate-limit 풀·공유 캐시")이 정확히 반영됐다. `쿼터 침범=0(WF별 token-bucket)`·`캐시 오염 전파=0`은 round-01에서 비어 있던 agentic 격리 차원을 직접 채운다. noisy-neighbor의 실체를 "쿼터 독점"으로 재정의하고, QA-03 runaway cap과 직접 연결(cap 없으면 한 WF가 공유 쿼터 독점)한 것도 정확. **닫혔다.**

- **잔여(silent cap, round-02 핵심 한계)**: 검증 전략이 솔직히 인정하듯 **외부 LLM 제공자의 rate-limit이 계정 전역이면 token-bucket은 client-side 분배만 보장**(제공자측 공유 한도 자체는 못 늘림). 즉 `쿼터 침범=0`은 *우리가 클라이언트에서 나눈 쿼터* 기준이지, 제공자 전역 429가 터지면 모든 WF가 동시에 굶는 건 격리로 못 막는다. → **QA-01 헤드룸과 동일 뿌리**: 헤드룸 ≥20%(QA-01)가 전역 풀에서 보장돼야 token-bucket 분배(QA-06)가 의미를 가진다. 두 QA cross-link 보강 권고(Low). 이건 시스템 본질적 한계라 신규 결함 아님 — 명시됨.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **C3(02↔06 경계 중복) 해소 확인**: round-01에서 QA-02와 QA-06이 둘 다 fault를 다뤄 중복이었다. 이번에 **QA-02 = 장애 단위의 복구 / QA-06 = 타 단위로의 격리**가 양쪽 정의에 `> altitude` 한 줄로 박혀 **양방향 cross-link 완료**(QA-02 검토에서도 확인). taxonomy 정리 완료. **C3 닫힘.**
- **KPI ○ 유지·강화**: round-01에서 이 세트 중 드물게 건강했던 ①②(중단 ≤1%·latency ≤10%)를 유지하고, ③④(쿼터·캐시오염)를 추가해 영향 채널(자원·상태·스케줄러)을 빠짐없이 덮었다. measurable·sound 모두 ○. **이 세트의 모범 사례.**
- consistency ○: QAS-06 자극(자원 폭주)·Measure 동기화 확인.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

이 QA는 렌즈3 처방(큐별 쿼터·token-bucket·bulkhead·circuit breaker)이 **KPI에 가장 직접 환원된 사례**다:
- `타 WF 중단 ≤1%`는 WF 타입/테넌트별 전용 큐·worker pool(bulkhead) + 큐별 max concurrency로, `쿼터 침범=0`은 공유 LLM 풀의 WF별 token-bucket rate limiter로 정확히 매핑. circuit breaker로 장애 WF 격리. **runner 메커니즘으로 직접 측정 가능 — 양호.**
- **잔여(DP 위임, OI-7)**: KPI를 **DP-0004 A5/A8 Bulkhead + DP-0005**에 귀속시키나, OI-7이 명시하듯 **DP-0004/0005가 쿼터·상태 격리를 보장하는지 미명시**다. 특히 **DP-0005 2안 공유 캐시 채택 시 ④(오염 전파 0)가 R-1 위험과 직접 충돌** — 무효화·읽기전용 계층화 명시가 없으면 KPI ④의 책임 설계가 *오히려 위험 요인*이다. DP 디스커션 위임(가장 명확한 DP 충돌 케이스).
- **runner 측 KPI**(재확인): 큐별 fair-queueing 위반율, token-bucket 분배 정확도, circuit breaker 발동·전파 차단율, 공유 캐시 무효화 지연(오염 검출→격리 시간).

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | ○ | **○** | 격리(blast-radius) 단일 관심사 + QA-02(복구) 경계 양방향 명문화로 C3 닫힘 |
| KPI가 측정 가능한가 | ○ | **○** | 건강 ①② 유지 + 쿼터/캐시오염 추가, 영향 3채널 빠짐없이 측정 |
| KPI가 현실적/적절한가 | ○ | **○** | token-bucket 분배 현실적. 단 외부 계정전역 rate-limit 한계는 silent cap(Low) |
| 정의↔KPI↔QAS 일치 | ○ | **○** | QAS-06 자극·Measure 동기화 확인 |

**verdict 변화: Sound ○→○ / KPI ○→○ · severity Med→Low.** round-01 Med의 근거(02↔06 경계·자원쿼터 누락)는 해소됨 — KPI는 이미 건강했고 보강만 추가. 잔여는 DP 역검토뿐.

## Stage 2 권고 (round-02)

이 세트의 모범 사례라 **DP 위임·cross-link만** 남긴다:

- **(DP 디스커션 위임, OI-7 — 가장 명확한 DP 충돌)** **DP-0005 2안 공유 캐시 채택 시 ④(오염 전파 0)와 R-1 위험이 직접 충돌** → 무효화·읽기전용 계층화를 DP-0005에 명시. DP-0004 bulkhead가 쿼터·상태 격리를 보장하는지 역검토.
- **(QA-01 cross-link, Low)** `쿼터 침범=0`(QA-06)이 의미를 가지려면 **QA-01 헤드룸 ≥20%가 전역 풀에서 보장**돼야 함을 cross-link으로 박음 — token-bucket 분배는 헤드룸이 있어야 작동.
- **(외부 한계 명문화, Low)** `쿼터 침범=0`이 **client-side 분배 기준**임을 명시 — 제공자 전역 429는 격리로 못 막음(silent cap).
- **(예시값 확정)** ③④의 `0`·캐시오염 임계는 격리 주입 시뮬으로 확정(①② 1%·10%는 건강 값 유지).
