# QA 리뷰 round-02 — 결론 리포트

> **이 문서 하나로 round-02의 전모를 파악할 수 있다.** 개별 근거는 `QA-0X-*.md`·`NQA-*.md`, 방법론은 [`../../Reviewer.md`](../../Reviewer.md).
> date: 2026-06-24 · scope: `context/qa/` QA 10 + NQA 3(신설) + QAS 13
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 직전 라운드: [round-01 review](../../round-01/review/report.md) → [counsel](../../round-01/counsel/counsel.md) → [contention(QA-09)](../../round-01/contention/counter.md) → [applier](../../round-01/applier/report.md)
> 절차 0(반영 보고서 재검증): applier report §3 "다음 Reviewer가 다시 볼 것"을 이번 라운드 우선 점검 목록으로 받아 전 항목 재판정함.

## TL;DR

- **round-01 [반영]이 효과적이었다.** High 4건(QA-01·07·08 + QA-02) 중 **3건이 닫혔고**(QA-01·02·08 → KPI ✕/△ → ○), **QA-07만 △로 잔존**(미채택 NQA-B 게이트 의존). KPI ✕ 3건은 **0건으로 소멸**.
- **남은 한 가지 구조 병목 = 신설 QA 미채택(OI-8).** QA-07 주 KPI·QA-09 ②-2가 **NQA-B에 닫힘 의존**, QA-01 활용률·QA-05 top-line이 **NQA-C에 부유**. **NQA-A/B/C 정식 채택이 세트 닫힘의 단일 트리거**다.
- **렌즈3 횡단 결론 = 다음은 DP 디스커션 차례.** round-02 잔여의 대부분은 "KPI는 측정가능해졌으나 그 KPI를 책임지는 DP가 비어 있다"(KPI-DP 귀속 placeholder, OI-7). 특히 **eval/검증 서브시스템이 어떤 DP에도 없는 신규 인프라**(NQA-B `related-dp: []`)라 DP 디스커션 1순위.
- **신규 결함 0건.** round-01 반영이 만든 *새 의존*(QA-07/09↔NQA-B)은 round-01 applier가 예측·트래킹한 것이라 신규 결함이 아니라 **예고된 미닫힘**이다.

## 종합 판정표 (round-01 → round-02 변화)

| QA | 속성 | R01 Sound/KPI · sev | R02 Sound/KPI · sev | 변화 핵심 |
|---|---|---|---|---|
| [QA-01](QA-01-scalability.md) | Scalability | △ / ✕ · **High** | ○ / ○ · **Low** | `N` placeholder 폐기 → scaling efficiency. **High 해소** |
| [QA-02](QA-02-availability.md) | Availability | ○ / △ · **High** | ○ / ○ · **Low** | MTTR 단독 → 4축 분해 + 외부 LLM 재개. **High 해소** |
| [QA-03](QA-03-controllability.md) | Controllability | ◎ / △ · Med | ◎ / △ · **Med** | C2 해소·runaway cap 편입. 단 ②위반0 측정수단 [발표 서사] 잔존 |
| [QA-04](QA-04-observability.md) | Observability | ○ / △ · Med | ○ / ○ · **Low** | 재구성95% 모호 → span 완전성·event-history. Med 해소 |
| [QA-05](QA-05-efficiency.md) | Efficiency | ○ / △ · Med | ○ / ○ · **Low** | 8k 폐기·캐싱 역페널티 제거. top-line→NQA-C(부유 의존) |
| [QA-06](QA-06-reliability-workflow.md) | Reliability(WF 격리) | ○ / ○ · Med | ○ / ○ · **Low** | C3 닫힘·쿼터/캐시오염 추가. 세트 모범. Med→Low |
| [QA-07](QA-07-performance-agent-time.md) | Performance(Agent) | △ / ✕ · **High** | ○ / △ · **Med** | "최대화" 폐기(High 해소). **주 KPI가 미채택 NQA-B 게이트 의존 → △ 잔존** |
| [QA-08](QA-08-performance-e2e.md) | Performance(E2E) | △ / ✕ · **High** | ○ / ○ · **Low** | 정의↔KPI mislabel 복구. **High 해소(가장 깔끔)** |
| [QA-09](QA-09-reliability-agent-consistency.md) | Reliability(일관성) | △ / △ · Med | ○ / △ · **Med** | **contention으로 헤드라인 Δ·②2단·H_norm 해소.** ②-2만 NQA-B 의존 △ |
| [QA-10](QA-10-maintainability.md) | Maintainability | ○ / ○ · Low | ○ / ○ · **Low** | 평균→p95·agentic 축. 건강 유지 |
| [NQA-A](NQA-A-security-safety.md) | Security/Safety | (신설·강력권장) | ○ / △ · **Med** | 정식화 권장. 4/5 KPI 측정수단 [발표 서사](공유 red-team 하네스) |
| [NQA-B](NQA-B-correctness.md) | Correctness | (신설·권장) | ○ / △ · **High** | 정식화 강력권장. **세트 닫힘 병목 — QA-07·QA-09(②-2) 닫힘이 NQA-B에 달림** |
| [NQA-C](NQA-C-cost-economy.md) | Cost-economy | (신설·권장 Med) | ○ / △ · **Med** | 정식화 권장. **미채택 시 QA-01/05 KPI 부유**(C2 닫힘 트리거) |

집계(13항목): **Sound ✕ 0 · KPI ✕ 0**(R01 KPI ✕ 3 → 0). **High 1(NQA-B) · Med 5(QA-03·07·09·NQA-A·C) · Low 7.**
기존 QA 10만 보면: R01 High 4·Med 5·Low 1 → **R02 High 0·Med 3·Low 7.**

## 교차(cross-cutting) 분석 — round-02

round-01의 C1~C5는 [applier report §2](../../round-01/applier/report.md)에서 처리됨을 재검증으로 확인했다(아래 "C 추적"). round-02는 **반영 후에도 남은 구조 문제**를 새로 번호 매긴다.

### C1 — 교차 의존 미닫힘 (QA-07·QA-09 ↔ NQA-B) **[sample QA-07이 식별]**
- **QA-07 주 KPI(first-pass 성공 판정)·QA-09 ②-2(정밀 유효-결정률)가 NQA-B golden 게이트에 닫힘 의존.** NQA-B가 미채택([이월], OI-8)·검증 [발표 서사]라 두 QA의 KPI가 독립 측정 불가 → 둘 다 △ 고정.
- applier §3-2 "교차 의존이 닫혔는가"의 답 = **아직 아니오.** 단 **NQA-B 정식 채택이 두 QA를 동반으로 닫는 단일 행동** — NQA-B가 세트 닫힘 병목(진원지, severity High).
- 비대칭: QA-09는 contention 덕에 ②-1(룰 게이트, golden 불요)로 *부분 닫힘* → QA-07(주 KPI 통째 의존)보다 닫힘도 높음.

### C2 — KPI 이양처 미채택 시 부유 (QA-01·QA-05 ↔ NQA-C) + KPI-DP 귀속 placeholder **[sample QA-07이 KPI-DP 측면 식별]**
- **(이양 부유)** QA-01 "자원 활용률"·QA-05 `$/완료모델`이 NQA-C로 이양됐는데 **NQA-C 미채택이면 두 KPI가 원 QA에도 NQA-C에도 안 살아 있는 부유 상태.** NQA-C 채택이 닫힘 트리거(severity Med — 발표 누락이지 측정 모순 아님).
- **(KPI-DP 귀속 placeholder, OI-7)** round-02 잔여의 *대부분*이 이 패턴: KPI는 측정가능해졌으나 **그 KPI를 책임지는 DP가 그 차원을 명시하지 않음.** 전 QA에 걸친 횡단 문제라 렌즈3 결론(아래)으로 승격.

### C3 — 검증의 [발표 서사] 집중 (eval 의존 KPI 군집)
- **공유 적대적/golden eval에 의존하는 KPI들이 통째로 [발표 서사]**: QA-03 ②(위반0)·NQA-A 4/5축·NQA-B 주 KPI·QA-09 ②-2. 이들은 **하나의 eval/검증 서브시스템(red-team + golden + judge + 결정 판정 게이트)을 공유**한다.
- 발표 방어선으로는 "검증하도록 *설계*했다"(모듈 박스 존치)가 유효하나, **acceptance 닫힘은 정식 채택 + 실측 전환이 전제**(OI-8). round-01 applier §3-1 "서사로 미룬 검증이 유효한 설계인가" 재판정 = **설계로는 유효, KPI 닫힘은 미완.**

### C 추적 — round-01 C1~C5 처리 확인
| R01 교차 | round-02 재검증 결과 |
|---|---|
| C1 측정불가 KPI(N·최대화) | **닫힘** — QA-01·07 placeholder 소멸(KPI ✕ → 0) |
| C2 정의↔KPI 불일치 | **닫힘** — QA-08 mislabel 복구·QA-03 SSoT 통일 |
| C3 QA 경계 중복(taxonomy) | **닫힘** — Performance 3분할·02↔06 경계 양방향 명문화 |
| C4 agentic 리스크 과소대표 | **KPI 정의는 닫힘 / eval 실행은 R02-C3로 이월**(rate-limit·runaway·trace·캐시우회는 반영, 실행은 [발표 서사]) |
| C5 누락 1급 QA | **신설 완료 / 정식 채택은 R02-C1·C2로 이월**(NQA-A/B/C 파일 존재, OI-8 대기) |

## 렌즈3(Runner 인프라) 횡단 결론 — 다음은 DP 디스커션

round-02 렌즈3의 단일 결론: **잔여의 대부분이 KPI-DP 귀속 placeholder(OI-7)다 — KPI는 측정가능해졌으나 그 KPI를 실현·측정하는 DP가 그 차원을 명시하지 않는다.** QA별 DP 위임 항목을 모으면:

- **rate-limit headroom·admission control·bounded queue** (QA-01·QA-06) → DP-0001/0004 미명시.
- **외부 LLM degradation backoff·폴백** (QA-02) → DP-0002/0003 미명시. + DP-0001 `drives: QA-02`→`QA-06` 교정.
- **runaway cap·graceful stop+롤백** (QA-03) → DP-0002/0003 미명시.
- **span 수준 trace 보장** (QA-04) → DP-0003 미명시("규칙 기반 한정" 시 span 범위).
- **비용 기준 라우팅** (QA-05·NQA-C) → DP-0001 토큰/비용 기준 미명시.
- **공유 캐시 오염 격리(무효화·읽기전용)** (QA-06) → DP-0005 2안 채택 시 R-1과 충돌.
- **first-pass 품질 게이트 hook** (QA-07) → DP-0001 latency만, 품질 게이트 무연결.
- **E2E latency 책임 + 5% 결정 산식 실측** (QA-08) → DP-0004/0005.
- **workflow 버저닝·계약 분리·blue-green** (QA-10) → DP-0004/0005 미명시.
- **보안 tactic(공급망 서명·secrets·injection 가드레일)** (NQA-A) → DP-0002/0003 미명시.
- **⭐ eval/검증 서브시스템 DP 신설** (NQA-B `related-dp: []`·NQA-A·QA-03·QA-07·QA-09 공유) → **어떤 DP에도 없는 신규 인프라. DP 디스커션 1순위.**

→ **QA 측 verdict는 이 항목들로 내려가지 않는다(QA 자체는 sound·measurable).** 전부 **DP로 넘겨야 할 항목**임을 확인 — applier §3-3과 일치, **다음은 DP 디스커션 차례.**

## Stage 2 착수 우선순위 (round-02)

1. **신규 QA 정식 채택(OI-8) — 최우선.** NQA-B(QA-07·QA-09 동반 닫힘 트리거) → NQA-A(QA-03 공유) → NQA-C(QA-01·QA-05 부유 닫힘). 채택 시 번호 재정렬·양방향 cross-link·INDEX/glossary 동기화 + **검증 [발표 서사]→실측 전환**.
2. **DP 디스커션 착수(OI-7).** 위 렌즈3 횡단 목록 — 특히 **eval/검증 서브시스템 DP 신설**이 NQA-B·NQA-A·QA-03·QA-07·QA-09를 한꺼번에 받친다.
3. **예시값 팀 합의.** 0.8·99.5%·6k·3배·6시간·$5·90% 등은 전부 "측정가능 KPI의 모양" 예시 — 발표 전 실측/모델/baseline로 확정(레퍼런스 복제 아님).
4. **importance 사람 결정.** QA-08(E2E top-line)·QA-09(NQA-B와 묶이면, contention 게이트 충족) 상향 여지.

## 이번 라운드에서 한 일 (요약)

- 절차 0: round-01 applier report §3 우선 점검 5항목을 전 QA 재판정에 반영(disposition 추적 — 각 파일 머리말에 명시).
- QA 10 + NQA 3을 3렌즈로 전수 재평가, round-01→round-02 verdict 변화를 4축 판정표로 기록.
- 교차분석 C1~C3(round-02) + round-01 C1~C5 처리 추적, 렌즈3 횡단으로 DP 디스커션 의제 도출.
- 신규 QA 후보: round-02 신규 발굴 0건(기존 NQA-A/B/C 정식화가 우선) — [`_new-qa-candidates.md`](_new-qa-candidates.md).

## 종료조건 신호 (수동 운영 참고)

- **신규 결함 0건** (반영이 만든 의존은 applier가 예고·트래킹한 것).
- **잔여 worklist는 전부 사람 결정(OI-8)·DP 디스커션(OI-7)으로 위임** — QA 측 자체 수정거리는 예시값 확정·Low 보강 cross-link뿐.
- → **QA 디스커션은 수렴 단계.** 다음 행동은 QA 라운드 N+1이 아니라 **(a) OI-8 채택 결정 (b) DP 디스커션 착수**. 신설 QA 채택 후엔 재번호·cross-link 검증용 1라운드가 필요할 수 있음.
