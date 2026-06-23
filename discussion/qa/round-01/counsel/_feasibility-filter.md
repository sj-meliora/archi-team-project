# Feasibility Filter — round-01 counsel 현실성 메타 평가

> 이 문서는 **새 권고를 만들지 않는다.** round-01 counsel(blue team)의 13개 권고를 **우리 팀(17조)의 실제 과제 조건**에 비춰 "현실적으로 채택 가능한가"로 필터링하는 메타 평가다. counsel 원문은 수정하지 않는다(append-only). 방법론·원문: [`counsel.md`](counsel.md) · [`_poc-plan.md`](_poc-plan.md)
> date: 2026-06-24

## 평가 기준선 (우리의 실제 조건)
1. **인증 과정 팀 과제.** 최종 산출물 = **발표 슬라이드(2026-07-07)** + 부속 문서·다이어그램. 동작하는 시스템이 아니다.
2. **대상 시스템은 가상 설계.** 러너·에이전트·파이프라인·GPU 풀이 **존재하지 않는다** → PoC를 **실측 실행할 수단이 없다.**
3. **팀 4명, 팀 점수는 공통·관대.** 인증 당락은 개인 과제에서 갈린다 → **과한 에포트는 명시적으로 지양.**
4. **가치 판단 기준**: "**문서/슬라이드의 설계 성숙도**에 기여하는가" vs "**실행 노동**을 요구하는가".

## 등급 rubric
- **[반영]** — 글쓰기만으로 끝나는 저비용 KPI/정의 교정. `context/qa/`에 실제 반영.
- **[발표 서사]** — PoC·검증을 "검증 방법 한 줄"로 압축해 슬라이드에만 싣는다(실행 0건).
- **[생략]** — 안 만들 시스템에만 의미 있는 실행 관리물. 드롭 또는 발표 1장으로 축소.
- 숫자(efficiency 0.8 · 1.8배 · pass^k 등)는 "증명 대상"이 아니라 **"측정가능 KPI를 이렇게 정의했다"는 예시**로 슬라이드에 둔다.

---

## 메인 태깅 표

| 항목 | counsel 핵심 권고 | 등급 | 우리 조건에서의 근거(1줄) | Stage 2 액션(한 줄) |
|---|---|:---:|---|---|
| **QA-01** Scalability | N·활용률 폐기 → scaling efficiency ≥0.8 + backlog 오토스케일 + rate-limit 헤드룸 | **[반영]** | 측정불가 placeholder(N) 제거는 순수 글쓰기 — 설계 성숙도에 직결 | KPI를 efficiency·헤드룸·큐 p95로 교체, 활용률 삭제 |
| **QA-02** Availability | MTTR 분해 → 재기동 ≤1분 AND 무손실 + 가용률 99.5% + 외부 LLM 장애 | **[반영]** | "무엇의 MTTR"·무손실·멱등은 정의 정밀화일 뿐 실행 불요 | KPI 4행으로 분해 기입(손실0/멱등100%/99.5%) |
| **QA-03** Controllability | QAS "위반 0건" 편입 + runaway cap + HITL + 적대적 eval | **[반영]** + 일부 [발표 서사] | QA↔QAS SSoT 통일·cap·HITL 명문화는 글쓰기. "적대적 eval로 검증" 부분만 서사 | KPI에 cap·HITL·위반0 편입 / 검증수단은 슬라이드 한 줄 |
| **QA-04** Observability | "재구성 95%" → trace 완전성(span 정의) + event-history + MTTD | **[반영]** | 모호한 "재구성"을 span 구성요소로 정의하는 건 설계 명료화 | KPI를 trace 완전성·event-history·MTTD로 환원 |
| **QA-05** Efficiency | top-line→NQA-C 승격, 캐시/신규 토큰 분리집계(8k 페널티 제거) | **[반영]** | 비현실 8k·캐시 역페널티 교정은 명백한 결함 수정 | KPI를 신규/캐시 분리·worker 가동률로 교정 |
| **QA-06** Reliability-WF | 건강 — 토큰/rate-limit 쿼터 격리 KPI 추가 + QA-02 경계 명문화 | **[반영]** | 기존 KPI 건강, 쿼터 격리 1행 추가 + cross-link만 | KPI③④(쿼터/캐시오염) 추가, QA-02↔QA-06 cross-link |
| **QA-07** Perf-Agent | "최대화" 폐기 → 노드타입별 speedup + first-pass 품질 게이트 + 커버리지 | **[반영]** | "최대화"는 acceptance 불가 → 비율·게이트로 교정은 핵심 교정 | KPI를 speedup·first-pass게이트·커버리지로 교체 |
| **QA-08** Perf-E2E | 정의↔KPI 정렬 → E2E latency p95·throughput 추가, 전달5% 강등 | **[반영]** | 정의(E2E)와 KPI(전달5%) 불일치는 치명적 정합성 오류 | KPI에 E2E latency p50/p95·throughput 추가, 전달5% 하위로 |
| **QA-09** Reliability-일관성 | 재프레이밍 → 캐시우회 pass^k + 유효-결정률, NQA-B와 짝 | **[반영]** + 일부 [발표 서사] | "일관≠정확"·캐시치트 지적은 정의 교정. pass^k 실측은 서사로 압축 | 정의를 유효-결정 안정성으로, KPI에 pass^k·유효결정률 / 측정은 서사 |
| **QA-10** Maintainability | 평균→CIS p95(tail) + prompt/model 교체·온보딩 축 추가 | **[반영]** | 평균→p95, model-agnostic 축 추가는 저비용 글쓰기 | KPI를 CIS p95 + prompt/model 교체·온보딩으로 확장 |
| **NQA-A** Security/Safety | red-team eval(위반0·injection) + HITL + 공급망 서명 | **[반영]**(신규 QA로 신설) | 자율 배포 권한의 공격면은 **설계상 1급 누락** — 아키텍처 요소로 남길 가치 큼 | 신규 QA 파일 신설(정의+KPI). red-team **실행**은 [생략] |
| **NQA-B** Correctness | golden set + LLM-as-judge 정답률 — QA-07/09 게이트 전제 | **[반영]**(신규 QA로 신설) | "맞는가"는 자율신뢰 #1 기둥 — 설계엔 +. golden set **구축**은 실행 부담 | 신규 QA 파일 신설(정의+정답률 KPI). golden/judge **구축**은 [생략] |
| **NQA-C** Cost-economy | $/완료모델 + 절감률 — QA-05 top-line·QA-01 활용률 흡수 | **[반영]**(신규 QA로 신설) | "수작업 대비 N% 절감"은 발표 ROI 핵심. KPI 정의는 글쓰기 | 신규 QA 파일 신설, QA-05/01에서 KPI 이동 |
| **PoC 전체(17개)** | 17 PoC 의존그래프·5~6주 일정·critical path·교차의존 orchestration | **[생략]** | 안 만들 시스템의 **실행 관리물** — 우리 조건상 무가치 | `_poc-plan.md`는 발표 1장("검증 전략 개요")으로만 축소 인용 |

### 등급 집계
- **[반영] 13건** (QA-01~10 + NQA-A/B/C). 단 QA-03·QA-09는 일부가 [발표 서사]로 분기.
- **[발표 서사] 3건** (QA-03 적대적 eval / QA-09 pass^k 실측 / 신규 QA들의 eval 하네스 검증 — 슬라이드 한 줄로).
- **[생략] 1건 + α** (`_poc-plan.md`의 17-PoC orchestration 전체. 부속으로 NQA-A red-team 실행·NQA-B golden set 구축 등 **개별 PoC의 "실행" 행위**).
- 핵심 분리 원칙: **KPI·정의 재설계 = [반영], 그것을 PoC로 "증명"하는 실행 = [생략]/[발표 서사].** 13개 권고의 글쓰기 부분은 거의 다 채택, 검증 노동은 거의 다 드롭.

---

## "실제로 손댈 것" — 최소 작업목록 ([반영]만, context/qa/ 반영)

> 모두 KPI/정의 문구 교정. 숫자는 placeholder가 아니라 "측정가능 KPI를 이렇게 정의했다"는 예시값으로 그대로 둔다.

1. **`QA-01-scalability.md`** — 정의에 "처리량 선형 유지, 1차 병목=LLM rate-limit, backlog 신호 구동" 추가. KPI: `완료모델 ≥N`·`활용률 ≥70%` 삭제 → `scaling efficiency ≥0.8(부하 2배→처리량 ≥1.8배)` + `rate-limit 헤드룸 ≥◯%` + `큐 대기 p95 ≤◯분`. (활용률은 NQA-C로 이전)
2. **`QA-02-availability.md`** — 정의에 "외부 LLM 장애 포함, 무손실 멱등 재개" 추가. KPI: `MTTR<1분` → `재기동 ≤1분 AND in-flight 손실=0` + `WF 성공률 ≥99.5%` + `외부 LLM 장애 자동재개율 ≥◯%` + `side-effect 멱등성=100%`.
3. **`QA-03-controllability.md`** — 정의에 "최소권한 allowlist + 고위험 HITL + runaway cap" 추가. KPI: `≤5초` → `ack ≤5초 AND graceful 정지+롤백 ≤◯초` + `허용범위 외 통과=0` + `runaway cap 작동 100%` + `HITL 통과율 100%·우회 0`.
4. **`QA-04-observability.md`** — 정의에 "span 캡처(prompt+context+tool I/O+model+token+ts), traces·metrics·alerting 3축". KPI: `재구성 ≥95%` → `trace 완전성 안전 100%/일반 ≥95%` + `event-history 100%` + `MTTD ≤◯분` + `결정당 비용/토큰 기록 100%`.
5. **`QA-05-efficiency.md`** — 정의에서 top-line을 NQA-C로 이양 명시. KPI: `총 토큰 ≤8k` → `신규 토큰 ≤◯ + 캐시 토큰 별도 집계(입력+출력 명시)` + tier표 유지 + `worker 가동률/task당 compute 비용`.
6. **`QA-06-reliability-workflow.md`** — 정의에 "자원 폭주·토큰/rate-limit 쿼터 격리(blast-radius)" 추가, QA-02(복구)와 구분 명시. KPI①②(중단≤1%/latency≤10%) 유지 + `쿼터 침범=0` + `공유 캐시 오염 전파=0` 추가. QA-02 cross-link.
7. **`QA-07-performance-agent-time.md`** — 정의에서 "최대화" 제거 → "1-pass 성공 노드 한정, 고정 노드집합, 노드타입별". KPI: `{사람−에이전트} 최대화` → `노드타입별 speedup ≥◯배` + `first-pass 성공분만 집계(품질 게이트)` + `커버리지 ≥◯%` + `runner 오버헤드 비율 ≤◯%`.
8. **`QA-08-performance-e2e.md`** — KPI에 `E2E latency/모델 ≤◯시간(p50/p95)` + `throughput ≥◯모델/일` 추가, 기존 `전달 ≤5%`는 하위 항목으로 강등 + `전달 성공률·무결성=100%` 추가. (정의↔KPI 정합성 복구)
9. **`QA-09-reliability-agent-consistency.md`** — 정의를 "유효(정책 허용) 결정 안정성"으로 재프레이밍(결정성 목표 폐기). KPI: `재현율 ≥80%` → `순수 추론 재현율(캐시 제외) ≥◯ (pass^k)` + `유효-결정률 ≥◯` + `재실행 분산 ≤◯`.
10. **`QA-10-maintainability.md`** — 정의에 "prompt/tool-def·LLM 모델 교체 내성, model-agnostic". KPI: `평균 CIS ≤2` → `CIS p95 ≤◯ + 컴포넌트 경계 정의` + `prompt/tool-def 수정 ≤◯` + `모델 교체 무중단` + `신규 모델 온보딩 ≤◯`.
11. **신규 `NQA-A-security-safety.md`** (context/qa/에 신설) — 정의 + KPI(권한상승/범위외 배포=0, 고위험 HITL 100%·우회0, artifact 서명 100%, injection 차단 ≥◯%, secrets 노출=0). 우선순위 상위로 재번호 후보.
12. **신규 `NQA-B-correctness.md`** (신설) — 정의 + KPI(golden 정답률 ≥◯%, rework율 ≤◯%, 회귀 미검출률 ≤◯%). QA-07/09 게이트의 전제로 cross-link.
13. **신규 `NQA-C-cost-economy.md`** (신설) — 정의 + KPI($/완료모델 ≤◯, 수작업 대비 절감률 ≥◯%, 비용 분해 토큰/compute/재시도). QA-05 top-line·QA-01 활용률 흡수 명시.

> 부수 작업(거의 0비용): QA 2자리 번호 재정렬(NQA-A/B 상위 진입), DP-0001~0005에 "rate-limit·admission control·runaway cap·외부 LLM degradation·model-agnostic 미명시" 한 줄 역검토 메모. → `context/open-issues.md`로 트래킹만.

---

## "버릴 것" — [생략] 명시

- **`_poc-plan.md`의 17-PoC orchestration 전체** (의존 그래프·3그룹 우선순위·5~6주 일정·critical path·공유 자산 재사용 계획): 동작 시스템이 없어 **실측 PoC가 0건**이므로 일정·의존 관리물은 무의미. 드롭하지 말되(append-only 원문 보존) **발표에는 1장으로만** — "이 KPI들은 부하시험/chaos/eval 하네스로 검증 설계됨"이라는 **검증 전략 개요** 슬라이드 1장으로 축소 인용.
- **개별 PoC의 "실행" 행위** (PoC-S1~M1, N-A1/B1/C1을 실제로 돌리는 것): efficiency 0.8·pass^k·정답률을 **측정·증명**하는 노동은 전부 생략. 숫자는 "이렇게 정의했다"는 예시로 슬라이드에만.
- **신규 eval 서브시스템의 실제 구축**: NQA-A red-team 세트(50~100건)·NQA-B golden set(단계별 50~100 사람 라벨)·LLM-as-judge 인간 일치도 검증은 **실행 부담**이므로 생략. 단 **"아키텍처 요소"로서의 eval/검증 서브시스템 박스는 모듈 다이어그램에 남길 가치 있음**(설계엔 +) — 구축 노동(−)과 분리.

> 분리 판정 요약: **설계 산출물(다이어그램의 박스·QA의 KPI 정의)에 남기는 것은 OK, 그것을 돌려서 수치를 뽑는 실행은 전부 드롭.**

---

## 발표 슬라이드용 한 줄 검증 서사 모음 ([발표 서사])

각 항목을 슬라이드에 "검증 방법 한 줄"로만 싣는다(실행 0건).

- **QA-01**: "부하 2배 투입 시 처리량 ≥1.8배(scaling efficiency ≥0.8)를 부하시험(k6/Locust)으로 검증하도록 설계."
- **QA-02**: "노드 강제 종료(chaos) 후 in-flight 손실 0·중복 배포 0·재개 ≤1분을 durable execution으로 검증하도록 설계."
- **QA-03 / NQA-A**: "권한 외 액션·prompt injection을 적대적 red-team eval 세트로 통과 0건 검증(제어성·보안 공유 하네스)."
- **QA-04**: "한 의사결정을 trace(OTel GenAI span)만으로 재구성 가능한지로 관측 완전성을 검증."
- **QA-05 / NQA-C**: "prompt caching On/Off A/B로 $/완료모델 절감을 측정해 '수작업 대비 N% 절감'을 입증하도록 설계."
- **QA-07 / NQA-B**: "노드타입별 speedup을 **first-pass 성공분에만** 집계(golden set 정답률 게이트)해 '빠르게 틀리기'를 차단하도록 검증."
- **QA-08**: "mock 4단계 파이프라인 부하시험으로 E2E latency p95·throughput·전달 오버헤드(A5 vs A8 5% 산식)를 측정하도록 설계."
- **QA-09**: "DP-0005 캐시를 **우회**해 동일 입력 k회 반복(pass^k)으로 순수 추론 일관성을 측정 — 캐시 100%는 치트임을 대조로 입증."
- **QA-10**: "대표 변경 시나리오(컴포넌트/prompt/모델 교체)로 CIS p95와 모델 교체 무중단을 측정하도록 설계."

> 공통 꼬리말 한 줄(발표용): "수치는 합격선 **정의 예시**이며, 실제 합격선은 실환경 측정으로 확정한다(레퍼런스 수치 복제 아님)."
