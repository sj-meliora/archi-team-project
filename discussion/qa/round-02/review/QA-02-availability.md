# Review: QA-02 Availability — 운영 안정성 (round-02 재평가)

> source: `context/qa/QA-02-availability.md` + `QAS-02-availability.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound ○ / KPI △ — High**
> verdict(round-02): **Sound ○ / KPI ○ — Low** — MTTR 단독 폐기·4축 분해로 측정가능성 회복(High 해소). 잔존은 외부 LLM degradation DP 미명시(OI-7)·예시값뿐 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-02 = **[반영]** (MTTR 단독 → 재기동≤1분·손실0 + 가용률 99.5% + 외부LLM 재개 + 멱등100%). 이월: DP-0002/0003 외부 LLM degradation(OI-7).

## 원문 요약 (반영 후)
- **정의**: 부분 장애 + **외부 LLM 제공자 장애**에도 진행 작업 무손실·멱등 재개로 연속성 유지. 가용성 1순위 = "빨리 재기동"이 아니라 무손실. QA-06(격리)과 경계를 `> altitude`로 분리.
- **KPI**: 주 = `재기동 ≤1분 AND in-flight 손실=0`(durable state). 보조 = `워크플로우 성공률 ≥99.5%(SLO)` · `외부 LLM 자동 재개율 ≥95%` · `side-effect 멱등성=100%`.
- **QAS-02**: 자극에 외부 LLM outage/429 추가, Measure 4축 동기화.

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01의 핵심 지적("진짜 위협은 내 노드 한 대가 아니라 외부 LLM 제공자")이 **정의·KPI·QAS 자극 세 곳 모두에 정착**했다. `외부 LLM 자동 재개율 ≥95%`(backoff+큐잉, 무한재시도 폭주 없이)는 round-01에서 비어 있던 agentic 지배 장애원을 직접 채운다. 이 지적은 닫혔다.

다만 **반영이 닫지 못한 agentic 잔여 두 가지**(신규 결함 아님 — 다음 단계 점검 지점):
- **외부 LLM degradation의 책임 DP가 비어 있다** — KPI ③(자동 재개율 95%)을 "책임지는 설계"가 검증 전략 표에서 스스로 인정하듯 *약하다*: DP-0003 2안 격리·3안 모니터링은 흡수 메커니즘이지 **outage/429 backoff·폴백 tactic을 명시하지 않는다**(OI-7). 즉 KPI는 측정가능하나 그 KPI를 달성한다는 설계 주장이 placeholder다 → **DP 디스커션 위임**. QA-01과 동일한 KPI-DP 단절 패턴(C2).
- **폴백 모델 전환의 일관성 누수** — outage 시 다른 모델로 재개하면 "멱등 재개"가 표면적으론 성공이나, **재개 후 결정이 원래 모델과 같은지**(결정 일관성)는 QA-09/NQA-B 영역으로 silent cap 처리됐다. 가용성 KPI는 "이어졌다"만 보고 "같은 답으로 이어졌나"는 안 본다 → QA-09와 cross-link 보강 권고(Low). 자동 재개율 95%가 *유효한* 재개인지 *아무 답으로나* 재개인지의 경계.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

round-01의 두 결함이 정확히 교정됐다:
- **`MTTR<1분` 단독(무엇의 MTTR? 미정의 + half-done compile 1분 복구는 비현실)** → 4축 분해. 특히 `재기동 ≤1분 AND in-flight 손실=0`이 대상(agent/노드)·조건(durable state)을 명시해 측정 가능. **measurable 합격.**
- **"자주 죽지만 빨리 복구"를 합격으로 통과시키는 함정** → `워크플로우 성공률 ≥99.5%(SLO)` 상위 지표로 차단. SRE error-budget 프레이밍이 옳게 들어왔다.

- **Sound**: round-01에서 이미 ○였고 유지. QA-06(격리)과의 경계가 정의에 `> altitude` 한 줄로 박혀 C3(02↔06 중복)가 닫혔다. 단 **양방향 cross-link이 아직 단방향**(QA-02 정의→QA-06 명시, QA-06→QA-02도 명시됨 확인 — 양방향 완료). consistency ○.

- **남은 형식 결함(경미)**: `related-dp`가 DP-0001 제거되며 `DP-0002, DP-0003`으로 조정됐으나, **OI-7이 명시하듯 DP-0001 파일은 여전히 `drives: QA-02`로 선언** → 단방향 불일치 잔존. 이건 QA-02 본문 문제가 아니라 DP 측 교정 사항(DP 디스커션) → verdict에 영향 없음, OI-7 트래킹.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

이 QA는 round-01 렌즈3의 핵심 처방(durable execution = event sourcing·체크포인트·replay)이 **가장 깔끔하게 KPI로 환원된 사례**다:
- `재기동 ≤1분 AND 손실=0`은 durable 워크플로우 엔진의 **lease timeout + 재스케줄 + at-least-once 전달 + 멱등 activity = exactly-once 효과**로 정확히 매핑된다. MTTR이 "lease 만료 + 재스케줄"로 환원되는 것은 측정 가능한 runner 메커니즘. **반영 양호.**
- 단, **외부 LLM degradation 흡수**(KPI ③)는 runner 측에서 `bounded queue + retry backoff + circuit breaker`로 실현되는데, 검증 전략은 이를 "fault-injection 모델"로 시연하겠다 하나 **그 메커니즘을 책임지는 DP가 비어 있다**(렌즈1과 동일 결론). bounded queue가 DP에 없으면 outage가 길 때 backpressure가 무한히 쌓여 재개율 정의 자체가 흔들린다.
- **runner 측 KPI**(재확인): failover 시간(lease timeout 파라미터), event-history replay 성공률(QA-04 공유), 외부 의존 circuit-breaker 발동률·outage 후 자동 재개 곡선, 멱등 키 충돌 0.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | ○ | **○** | 무손실·멱등 복구 단일 관심사 유지 + QA-06 경계 양방향 명문화 |
| KPI가 측정 가능한가 | △ | **○** | `MTTR<1분` 단독 폐기 → 4축 구체 임계값. acceptance test 작성 가능 |
| KPI가 현실적/적절한가 | △ | **○** | 외부 LLM 장애 자동 재개율로 지배 장애원 반영. 단 폴백 일관성·degradation DP는 Low 보강 |
| 정의↔KPI↔QAS 일치 | ○ | **○** | QAS-02 자극(외부 LLM)·Measure 4축 동기화 확인 |

**verdict 변화: Sound ○→○ / KPI △→○ · severity High→Low.** round-01 High의 근거(MTTR 불완전 + 외부 장애 시나리오 부재)는 모두 해소됨.

## Stage 2 권고 (round-02)

대부분 닫혔으므로 **잔여 보강(Low)·DP 위임만** 남긴다:

- **(DP 디스커션 위임, OI-7 — 최우선 잔여)** DP-0002(Standby)·DP-0003(격리/모니터링)에 **외부 LLM degradation(outage/429) backoff·폴백 tactic + bounded queue**를 명시. KPI ③(재개율 95%)의 "책임지는 설계"가 현재 가장 약한 고리. 또 **DP-0001의 `drives: QA-02` → `QA-06` 교정**(OI-7).
- **(폴백 일관성 cross-link, Low)** `외부 LLM 자동 재개율 ≥95%` 옆에 "재개가 *유효한* 결정으로 이어졌는지는 QA-09/NQA-B에서 본다"는 silent cap을 cross-link으로 명시 — 자동 재개율이 *아무 답으로나* 재개를 합격으로 세지 않게.
- **(예시값 확정)** `1분·99.5%·95%·100%`는 chaos/fault-injection 모델·실환경 측정으로 확정(round-01부터의 미결 — 발표 전 팀 합의).
