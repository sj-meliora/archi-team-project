# QA 권고 round-02 — 종합 보고서 (Council / blue team)

> 이 문서 하나로 round-02 **권고의 전모**를 파악한다(공의회 최종 문서). 방법론: [`../../Council.md`](../../Council.md) · red team 결론: [`../review/report.md`](../review/report.md) · PoC 통합: [`_poc-plan.md`](_poc-plan.md)
> date: 2026-06-24 · scope: **full**(13항목 + 종합 + _poc-plan 완성) · seats: (1) Agentic Workflow · (2) 수석 아키텍트 · (3) Runner 인프라 · 직전 라운드: [round-01 counsel](../../round-01/counsel/counsel.md)

## TL;DR

round-02 review 핵심 = **수렴 단계**(KPI ✕ 0·신규 결함 0). Council의 임무는 *새 KPI 발명이 아니라 닫힘 메커니즘 확정·채택 근거 강화·동반 닫힘 트리거 명시*였다. 13항목 종합 결론:

- **닫힘 확인 7건**(QA-01·02·04·05·06·08·10) — KPI ○로 닫혔고 잔여는 전부 **비-verdict**(예시값 확정·Low cross-link·DP 위임). Council은 닫힘을 확인하고 보강만 권고.
- **채택 권장 3건**(NQA-A·B·C) — 셋 다 Sound ○·ISO 앵커 충분 → **정식 1급 QA 자격**. KPI △는 측정 수단([발표 서사]·이양처)이 *채택 동반*으로 닫히는 구조라 정식화 자체가 닫힘 행동(OI-8 사람 결정).
- **Med 잔여 2건**(QA-03·07·09) — 자체 결함 아님. **NQA-A/B 채택 시 동반 닫힘**(QA-03↔NQA-A red-team, QA-07·QA-09 ②-2↔NQA-B golden).
- **잔여 worklist는 두 갈래 위임**: (1) **신규 QA 채택 OI-8**(사람 결정 — PoC가 대체 못함), (2) **KPI-DP 귀속 placeholder OI-7**(DP 디스커션 차례). actionable [반영]거리는 예시값·Low cross-link뿐.
- **단일 eval/검증 DP로 수렴**(C3): NQA-A red-team·NQA-B golden·QA-03 ②·QA-07 first-pass 게이트·QA-09 결정 판정 게이트가 **하나의 eval/검증 서브시스템**을 공유 → DP 디스커션 1순위 의제.

## 채택 권고표

| QA | 속성 | review verdict | stance | 핵심 권고(한 줄) | PoC |
|---|---|:---:|---|---|---|
| [QA-01](QA-01-scalability.md) | Scalability | KPI ○ / Low | 닫힘 확인 | 닫힘 + 부하 단위·헤드룸 단위 명시(Low) + NQA-C 동반 채택·rate-limit DP 위임 | PoC-S1·S2 |
| [QA-02](QA-02-availability.md) | Availability | KPI ○ / Low | 닫힘 확인 | 닫힘(4축 무손실) + 외부 LLM degradation DP·`drives` 교정 위임 | PoC-A1·A2 |
| [QA-03](QA-03-controllability.md) | Controllability | KPI △ / Med | 조건부(NQA-A 동반) | ② 위반0을 **NQA-A 공유 red-team 하네스로 실측 전환** + 커버리지/내부롤백 명문화 + cap DP 위임 | PoC-C2≡N-A1 |
| [QA-04](QA-04-observability.md) | Observability | KPI ○ / Low | 닫힘 확인 | 닫힘(span 환원) + **다수 QA 측정 토대** 강조 + DP-0003 span 보장 위임 | PoC-O1 |
| [QA-05](QA-05-efficiency.md) | Efficiency | KPI ○ / Low | 닫힘 확인 | 닫힘(캐시 분리) + 캐시 적중률 보조 노출(Low) + NQA-C 동반·비용 라우팅 DP 위임 | PoC-E1 |
| [QA-06](QA-06-reliability-workflow.md) | Reliability-WF | KPI ○ / Low | 닫힘 확인 | 닫힘(C3·세트 모범) + QA-01 헤드룸 cross-link + 격리 DP 역검토 위임 | PoC-R1 |
| [QA-07](QA-07-performance-agent-time.md) | Perf-Agent | KPI △ / Med | 조건부(NQA-B 동반) | 주 KPI 분모(성공 판정)가 **NQA-B golden 의존** → 채택이 닫힘 전제 + baseline 프로토콜 명시(Med) | PoC-P1 |
| [QA-08](QA-08-performance-e2e.md) | Perf-E2E | KPI ○ / Low | 닫힘 확인 | 닫힘(가장 깔끔) + DP-0004 5% 산식 실측·importance 상향 여지(사람 결정) | PoC-E2E1 |
| [QA-09](QA-09-reliability-agent-consistency.md) | Reliability-일관성 | KPI △ / Med | 조건부(②-2만 NQA-B) | contention으로 **부분 닫힘**(Δ·②-1·H_norm ○) + ②-2만 NQA-B 의존 + importance 사람 결정 | PoC-K1 |
| [QA-10](QA-10-maintainability.md) | Maintainability | KPI ○ / Low | 닫힘 확인 | 닫힘(건강 유지) + 컴포넌트 경계 정의 전제 명시 + 교체내성 DP 역검토 위임 | PoC-M1 |
| [NQA-A](NQA-A-security-safety.md) | Security/Safety | KPI △ / Med | **정식 채택 권장(상위 진입)** | red-team 4축 실측 전환 + **QA-03 ② 공유 하네스 동반 닫힘**(C3) + 보안 tactic DP 위임 | PoC-N-A1≡C2 |
| [NQA-B](NQA-B-correctness.md) | Correctness | **KPI △ / High** | **정식 채택 강력 권장(OI-8 최우선)** | golden+judge 실측 전환 + judge 일치도 SLI 추가 → **QA-07·QA-09 ②-2 동반 닫힘의 단일 트리거** | PoC-N-B1(R2 확장) |
| [NQA-C](NQA-C-cost-economy.md) | Cost-economy | KPI △ / Med | **정식 채택 권장(Med)** | $/완료모델 분해 + baseline 가정 슬라이드 명시 → **QA-01 활용률·QA-05 top-line 부유 해소**(C2) | PoC-N-C1 |

집계(13항목): **닫힘 확인 7**(QA-01·02·04·05·06·08·10) · **채택 권장 3**(NQA-A·B·C) · **조건부(동반 닫힘) 3**(QA-03·07·09). 신규 KPI 발명 0건 — round-02는 닫힘 메커니즘 확정 라운드.

## 핵심 근거맵 (§4 라이브러리 유형 → 적용 QA)

| §4 유형 | 적용 QA | 라운드 핵심 |
|---|---|---|
| 정확성(golden·LLM-as-judge) | NQA-B(허브) · QA-07 게이트 · QA-09 ②-2 | judge 일치도 SLI 추가, **세트 닫힘 트리거** |
| 제어·안전(적대적 eval) | NQA-A · QA-03 ② | **공유 red-team 하네스**(OWASP LLM Top-10·SLSA·ISO Security) |
| 일관성(pass^k) | QA-09 | contention으로 Δ 대조·H_norm·② 2단 — **부분 닫힘** |
| 확장성(USL·backlog) | QA-01 | 부하 단위 고정·rate-limit 헤드룸 |
| 가용성·복구(SLO·durable) | QA-02 | 4축 무손실·외부 LLM 재개 |
| 지연(percentile) | QA-07·QA-08·QA-10(CIS p95) | 평균 금지·p95 |
| 비용·효율(prompt caching) | QA-05 · NQA-C | 캐시 분리·$/완료모델 분해 |
| 관측성(OTel GenAI) | QA-04(토대) → 비용·성능 수급 | span 완전성·`gen_ai.usage.*` 허브 |
| 격리(bulkhead·token-bucket) | QA-06 | 쿼터 침범0·캐시 오염0 |

## 교차권고 (red team C1~C3 round-02에 대한 응답)

- **C1 교차 의존 미닫힘 (QA-07·QA-09 ↔ NQA-B) → NQA-B 정식 채택이 단일 트리거.** NQA-B golden 게이트가 QA-07 주 KPI(first-pass 성공 판정 — 분모 통째 의존)와 QA-09 ②-2(정밀 유효-결정률 — ②-1로 부분 닫힘이라 ②-2만 의존)를 **동반 닫는다.** PoC 레벨에서 PoC-N-B1 출력이 PoC-P1·K1에 공급됨을 [`_poc-plan.md`](_poc-plan.md)에서 의존그래프로 명시. **닫힘도 비대칭: QA-07 > QA-09**(NQA-B 채택 우선순위 효과가 QA-07에서 가장 큼).
- **C2 KPI 이양처 미채택 부유 (QA-01·05 ↔ NQA-C) → NQA-C 정식 채택이 닫힘 트리거.** QA-01 "자원 활용률"·QA-05 `$/완료모델` top-line이 NQA-C로 이양됐는데 NQA-C 미채택이면 두 KPI가 원 QA에도 NQA-C에도 안 살아 있는 **부유 상태**. NQA-C 채택(이양처 확정) + QA-01/05 양방향 cross-link이 닫힘. NQA-B(게이트)와 달리 NQA-C는 *이양처*라 severity Med(발표 누락이지 측정 모순 아님).
- **C3 검증의 [발표 서사] 집중 → 단일 eval/검증 서브시스템 DP로 수렴.** NQA-A red-team·NQA-B golden·QA-03 ②·QA-07 first-pass 게이트·QA-09 결정 판정 게이트가 **하나의 eval/검증 서브시스템**(red-team + golden + judge + 결정 판정 게이트)을 공유한다. round-02 counsel은 5개 항목의 PoC를 **공유 인프라 2개**(red-team 하네스: QA-03≡NQA-A / golden 게이트: NQA-B→QA-07·QA-09·NQA-C)로 묶어, 이들이 **단일 DP로 귀결됨**을 확인. → **OI-7의 큰 줄기 = "eval/검증 서브시스템 DP 신설"**, DP 디스커션 1순위. QA 측에선 verdict로 내리지 않고 DP 위임으로 정리(C2 방침).

## PoC 로드맵 요약 (상세 [`_poc-plan.md`](_poc-plan.md))

round-02는 **신규 PoC가 아니라 round-01 PoC 중 [발표 서사]로 미룬 것의 실측 전환**이 핵심. 허브 2개:
1. **PoC-N-B1(golden 게이트)** — QA-07 PoC-P1·QA-09 PoC-K1(②-2)·NQA-C PoC-N-C1(성공분 비용)에 출력 공급. 정확성 닫힘이 곧 세트 닫힘.
2. **공유 red-team 하네스(PoC-C2 ≡ N-A1)** — QA-03 ②·NQA-A 4축을 동일 하네스 1개로 동반 산출.
- 토대: **PoC-O1(OTel 계측)**이 비용·성능 PoC에 토큰 계측 공급(QA-04 → NQA-C·QA-05·QA-07 오버헤드).
- **한계(전역 silent cap)**: 동작 시스템 부재로 round-02 실측은 0건 — 본 PoC 계획은 **"검증 전략 개요"**(발표용)로, 합격선 수치(◯)는 PoC 실행 시 확정.

## 직전 라운드(round-01) 대비 변화

| 축 | round-01 | round-02 |
|---|---|---|
| KPI ✕(측정불가) | 3건(QA-01·07·08 N·최대화) | **0건**(전부 소멸) |
| High | 4(기존 QA) | **1**(NQA-B, 세트 병목) |
| counsel 성격 | KPI 측정가능화(✕→측정형식) | **채택 트리거·DP 위임**(측정→닫힘) |
| 신규 KPI 발의 | 다수 | **0**(닫힘 메커니즘 확정만) |
| 신규 QA | NQA-A/B/C 신설 권고 | 정식화(OI-8) 권고 + 동반 닫힘 명시 |

권고 성격이 *측정가능화*(round-01) → *채택 트리거·DP 위임*(round-02)으로 이동. 이것이 **수렴 신호**다.

## 미해결 / 추가조사 (팀 결정·silent cap)

- **OI-8 정식 채택은 사람 결정** — PoC는 채택을 대체하지 못한다(NQA-A/B/C counsel 공통 silent cap). 채택 시 QA 2자리 번호 재정렬·INDEX/glossary 동기화·양방향 cross-link 확정. 우선순위: **NQA-B(최우선·세트 병목) → NQA-A(QA-03 공유·상위 진입) → NQA-C(QA-01/05 부유 해소·Med)**.
- **OI-7 eval/검증 서브시스템 DP 부재** — QA 측 verdict는 내려가지 않고 **DP 디스커션으로 위임(1순위)**. C3 수렴 결과 = 단일 DP. 그 외 QA별 DP 위임(rate-limit headroom·외부 LLM degradation·runaway cap·비용 라우팅·교체내성)도 DP 디스커션 의제.
- **예시값 실측 확정** — `90%·10%·5%·3배·0.8·99.5%·6시간·$5·Δ·H_norm 0.2·일치도` 등은 전부 "측정 가능 KPI의 모양" 예시값(레퍼런스 복제 아님). golden/baseline/SLO/단가 실측으로 확정 — 발표 전 팀 합의.
- **importance 사람 결정** — QA-08(E2E top-line)·QA-09(NQA-B 묶이면, contention 게이트 충족)·NQA-A/B(신뢰 두 기둥) 상향 여지. verdict 무관 우선순위 결정.
- **동작 시스템 부재 = 실측 0건** — 본 라운드 PoC는 검증 전략 개요(설계 산출물). "검증하도록 설계했다"가 발표 방어선, acceptance 닫힘은 채택+실측 전제.

## 종료조건 평가 신호 (수동 운영 참고)

- **actionable [반영]거리(QA 라운드 N+1 자체 수정)** = 예시값 확정 + Low cross-link(QA-01 헤드룸 단위·QA-05 캐시 적중률·QA-06 외부 rate-limit·QA-10 경계 정의)뿐 — **소진 임박**.
- **위임거리(다른 디스커션·사람)** = OI-8 채택(사람) + OI-7 DP 디스커션(특히 eval/검증 서브시스템 단일 DP) — **대부분이 여기**.
- → **QA 디스커션은 수렴.** 다음 행동은 QA 라운드 N+1이 아니라 **(a) OI-8 채택 결정 (b) DP 디스커션 착수.** 단, **신설 QA 채택 후 재번호·양방향 cross-link 검증용 1라운드**는 필요할 수 있음(append-only 추적).
