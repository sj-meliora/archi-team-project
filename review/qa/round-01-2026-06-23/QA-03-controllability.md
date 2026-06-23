# Review: QA-03 Controllability — Agent 제어 용이성

> source: `context/qa/QA-03-controllability.md` + `QAS-03-controllability-stop.md`
> verdict: **Sound ◎ / KPI △** — 가장 중요한 agentic QA. 품질속성은 우수하나 KPI가 불완전·QAS와 불일치 · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: Agent가 허용 범위 내에서만 동작.
- **KPI(QA 파일)**: 중단 명령 수용 시간 ≤ 5초
- **KPI(QAS-03)**: 중단 수용 ≤ 5초 **+ 허용 범위 외 액션 통과 0건** ← QA 파일엔 누락
- **QAS**: 자극=stop 명령 발행 / Agent가 허용범위 밖 액션 시도 / 응답=즉시 중단 / 차단.

## 렌즈 1 — Agentic Workflow 전문가 관점

**이 QA는 자율 시스템의 정의적(defining) 품질속성이다 — steerability / human oversight.** 업계가 자율 에이전트에서 가장 먼저 요구하는 속성이며, 이 과제의 “사람 개입 없이”라는 전제를 안전하게 만드는 유일한 안전장치다. 방향은 정확하다. 다만 세 가지가 빠졌다:
- **최소권한·도구 allowlist·HITL 게이트**: deploy/delete 같은 고위험 액션은 “차단”을 넘어 **사람 승인 게이트**가 정석. “허용 범위”의 정의 자체가 권한 모델이어야 한다.
- **Runaway loop 방지**: 자율 에이전트의 대표 실패는 무한 재시도/루프다. **max iteration·token budget·wall-clock cap 도달 시 자동 중단**이 제어성의 필수 축인데 빠져 있다.
- **검증 수단 부재**: “허용 범위 외 0건”을 무엇으로 증명하나? **적대적 eval/red-team 세트**가 없으면 0건은 측정 불가능한 구호다.
- **stop 의미 정의**: “수용(ack) ≤5초”와 “실제 정지”는 다르다. LLM 생성 중·deploy 중 5초 ack가 곧 안전 정지는 아니다 → graceful stop(롤백) vs hard kill을 구분해야.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **QA↔QAS 불일치(C2)**: QAS-03엔 “허용범위 외 0건”이 있는데 QA-03 파일 KPI엔 “≤5초”만. SSoT가 갈라져 있다 → 통일.
- **두 관심사 결합**: (a) 중단 가능성(stop latency)과 (b) 권한 집행(violation rate)은 다른 메커니즘이다. 한 QA로 묶는 건 허용되나 **KPI를 둘 다** 명시해야 한다.
- **측정 정의 필요**: “수용 ≤5초”의 수용은 ack인가 정지완료인가? “0건”의 분모(테스트 모집단)와 측정 방법은? 정의 없으면 테스트 불가.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**중단과 권한은 runner의 control plane 기능으로 구현된다.**
- **협조적 취소(cooperative cancellation)**: 오케스트레이터가 cancel 신호를 보내고, 장기 activity는 체크포인트마다 cancel token을 폴링해 안전 중단. “≤5초”를 만족하려면 장기 작업에 **취소 폴링 지점**을 심고, 불응 시 **hard-kill(pod 강제 종료) 폴백**을 둔다 — “수용”과 “정지완료”가 다르다는 렌즈1·2 지적의 구현적 근거.
- **dispatch 전 admission gate**: 권한 체크는 control plane에서 **실행 전에**(값싸게) 수행 — 토큰을 안 쓰고 차단(QAS-03 DP-0003 게이트와 일치).
- **runaway 방지는 runner 1급 기능**: activity timeout·최대 retry·max wall-clock을 runner가 강제(Temporal류 timeout/retry policy). cap 도달 시 자동 종료 → 렌즈1의 runaway cap을 인프라로 보장.
- **runner 측 KPI**: cancel 전파 지연 p95, timeout/retry cap이 설정된 activity 비율 100%.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | ◎ | 자율 시스템 1급 QA. 안전·통제의 핵심 |
| KPI가 측정 가능한가 | △ | ≤5초는 측정 가능하나 “수용” 정의 모호; “0건”은 검증수단 부재 |
| KPI가 현실적/적절한가 | △ | runaway cap·HITL·최소권한 축 누락 |
| 정의↔KPI↔QAS 일치 | △ | QA 파일이 QAS의 0건 KPI를 누락 |

## Stage 2 권고

- **KPI 통일·확장**:
  - ① 중단 수용 ≤ 5초 (수용=ack 정의, 별도로 **graceful 정지+롤백 완료 ≤ ◯초** 명시)
  - ② **허용 범위 외 액션 통과 = 0** (검증: 적대적 eval N건 중 0 통과) ← QA 파일에 추가
  - ③ **Runaway cap**: max iteration/token/wall-clock 도달 시 자동 중단 100%
- **권한 모델 명문화**: 최소권한 + 도구 allowlist + 고위험 액션 HITL 게이트(통과율·우회 0건).
- **C5 연계**: 이 QA는 신규 Security QA(NQA-A)와 인접 — 제어성은 안전의 부분집합. 경계 정리.
- DP 연결 점검: DP-0002(단일 제어지점)·DP-0003(권한 게이트)이 runaway cap·HITL을 포함하는지 역검토.
