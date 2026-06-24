# Review: QA-03 Controllability — Agent 제어 용이성 (round-02 재평가)

> source: `context/qa/QA-03-controllability.md` + `QAS-03-controllability-stop.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound ◎ / KPI △ — Med**
> verdict(round-02): **Sound ◎ / KPI △ — Med** — KPI 통일·4축 확장으로 측정구조는 개선됐으나, **주변 KPI(위반0)의 검증이 [발표 서사]로 미뤄져** 닫힘 미완 + runaway cap·graceful 정지 DP 미명시(OI-7) · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-03 = **[반영] + 일부 [발표 서사]** (수용≤5초 → ack/안전정지 분리 + 위반0·runaway cap·HITL). 적대적 eval 실측 [발표 서사]; DP runaway cap(OI-7).

## 원문 요약 (반영 후)
- **정의**: 최소권한·도구 allowlist + 고위험 액션 HITL 게이트 + 중단·runaway cap 즉응. NQA-A(Security)와 `Controllability ⊂ Security` 경계 명시.
- **KPI 4축**: 주 = `중단 ack ≤5초 AND 안전 정지(graceful stop)+롤백 ≤30초`. 보조 = `허용범위 외 통과=0(적대적 eval)` · `runaway cap 작동=100%` · `HITL 통과율=100%·우회=0`.
- **QAS-03**: 자극에 runaway·고위험 액션 추가, Measure 4축 동기화.

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01의 가장 중요한 두 지적이 반영됐다:
- **stop 의미 분리**(ack ≠ 실제 정지) → `ack ≤5초 AND graceful stop+롤백 ≤30초`. 생성 중·배포 중 5초 ack가 안전 정지가 아니라는 핵심 함정을 KPI 구조로 분리했다. **양호.**
- **runaway cap**(max iter·token·wall-clock) → KPI ③로 정식 편입. 자율 에이전트의 대표 실패(폭주)를 1급 제어 축으로 올렸다.

그러나 **round-02의 핵심 쟁점 — 주변 KPI의 검증이 [발표 서사]에 묶여 있다:**
- KPI ②(`허용범위 외 통과=0`)의 측정 수단 = **적대적 eval N건**인데, 그 eval 세트 구축·실행은 [발표 서사]/[생략]이다(NQA-A red-team 하네스와 공유). 즉 "위반 0건"은 **측정 수단이 살아 있지 않은 KPI**다. round-01 applier report §3-1이 "발표 서사로 미룬 검증이 여전히 유효한 설계인가"라 물은 바로 그 항목. → **재판정 결과: 설계 서사로는 유효하나(red-team 하네스를 모듈 박스로 존치), "0건"의 acceptance가 실측으로 닫히지 않아 KPI는 △ 유지.** 발표 방어선으로는 충분(아키텍처 산출물=OK), QA 완성도로는 미닫힘.
- **적대적 eval 커버리지가 곧 신뢰 상한**(미상상 우회 미검출)이 silent cap으로만 있다 — "0건"이 *우리가 상상한 공격에 대해* 0건임을 KPI 정의에 명문화 권고(과대주장 방지).

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **C2(QA↔QAS 불일치) 해소 확인**: round-01에서 QAS-03엔 있던 "허용범위 외 통과 0건"이 QA-03 파일 KPI엔 없어 SSoT가 분기했었다. 이번에 QA 파일 KPI ②로 정식 편입돼 **양쪽이 일치**. consistency ○.
- **Sound ◎ 유지**: 제어성은 이 과제("사람 개입 없이")를 안전하게 만드는 핵심 안전장치로 단일 관심사 명확. NQA-A와 `⊂` 경계도 명문화돼 중복 없음. round-01의 ◎가 정당하게 유지된다.
- **KPI △ 유지 사유**: ② 위반0의 측정수단([발표 서사])이 닫히지 않음(렌즈1). 그 외 ①ack/정지, ③runaway, ④HITL은 측정 구조가 잘 짜였으나 ③·④의 *작동 100%* 역시 시연 모델 의존(보조 모델은 제시됨). **High는 아님**(완전 측정 불가가 아니라 일부 측정수단 미실현) — Med 적정.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **협조적 취소 + hard-kill 폴백**(round-01 렌즈3 처방) → `ack/graceful stop` KPI + 검증 전략의 "cancel 폴링 밀도별 ack·정지완료 분포" 모델로 정확히 환원됐다. activity timeout/retry cap으로 runner가 cap을 강제하는 구조도 KPI ③에 반영. **반영 양호.**
- **잔여(DP 위임, OI-7)**: 검증 전략 KPI ③이 "DP-0002/0003이 runaway cap을 명시 안 함"을 스스로 인정한다. **graceful stop+롤백(협조적 취소 vs hard-kill)·runaway cap이 DP-0002/0003 후보 대안·ATAM에 없다**(OI-7). DP-0002 1안 HITL·DP-0003 1안 allowlist는 있으나 cap·정지 의미 분리가 없음 → KPI 핵심 장치의 설계 귀속이 비어 있음. DP 디스커션 위임.
- **부분 롤백 정합성**: deploy 중 취소 시 외부 시스템(Jira·빌드서버) 상태의 부분 롤백 정합성이 silent cap. graceful stop 30초가 *외부 부작용까지* 되돌리는지는 통합 환경 필요 → KPI에 "내부 상태 기준"임을 명시 권고(Low).
- **runner 측 KPI**(재확인): cancel 전파 지연 p95, hard-kill 폴백 발동률, cap별(iter/token/wall-clock) 차단 정확도, HITL 게이트 우회 탐색 커버리지.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | ◎ | **◎** | 제어성 단일 관심사 + NQA-A `⊂` 경계 명문화. 과제 전제를 안전화하는 1급 |
| KPI가 측정 가능한가 | △ | **△** | ①ack/정지·③cap·④HITL은 측정구조 양호하나, ②`위반0`의 수단(적대적 eval)이 [발표 서사]로 미실현 |
| KPI가 현실적/적절한가 | △ | **○** | runaway cap·graceful stop으로 자율 폭주·강제종료 함정 차단. 단 부분 롤백·커버리지는 Low 보강 |
| 정의↔KPI↔QAS 일치 | △ | **○** | C2 해소 — QAS의 "위반0"이 QA 파일 KPI로 정식 편입, 양쪽 일치 확인 |

**verdict 변화: Sound ◎→◎ / KPI △→△ · severity Med→Med.** 측정구조·consistency는 개선(현실성 △→○, 일치 △→○)됐으나, **②의 측정수단이 [발표 서사]에 묶여** KPI 종합은 △에 머문다. round-01 applier §3-1의 "서사로 미룬 검증" 재판정 = **설계 서사로는 방어 가능, KPI 닫힘은 미완.**

## Stage 2 권고 (round-02)

- **(서사 vs 닫힘 결정, OI-8)** ②`위반0`을 KPI로 유지하려면 **NQA-A 정식 채택 + 공유 red-team 하네스 실측 전환**이 선결(OI-8). 미채택·미실측이면 ②는 "발표 서사 지표"로 명시 강등 권고(닫힌 acceptance가 아님을 솔직히). round-01 applier §4 "주 KPI 선택" 절충과 동일 트랙.
- **(DP 디스커션 위임, OI-7)** DP-0002/0003에 **runaway cap·graceful stop+롤백 tactic** 명시. KPI ③·①의 설계 귀속이 비어 있음.
- **(커버리지 한계 명문화, Low)** ②`허용범위 외 통과=0` `기존(절대 0)` → `제안: 적대적 eval 세트 기준 0건(세트 커버리지가 신뢰 상한, OWASP LLM Top-10 매핑 log)`. 과대주장 방지.
- **(부분 롤백 범위 명시, Low)** graceful stop+롤백 30초가 **내부 상태 기준**임을 명시, 외부 시스템 부작용 롤백은 silent cap.
- **(예시값 확정)** `5초·30초·0건·100%`는 취소 타이밍 모델·실환경 측정으로 확정.
