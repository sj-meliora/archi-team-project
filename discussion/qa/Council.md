# Council — QA 개선 권고 재생산 스펙 (blue team)

> 목적: Council(자문 합의체) 에이전트가 **Reviewer(red team)의 지적을 전부 읽고, QA를 어떻게 바꾸면 그 약점을 극복하고 더 나은 QA set이 되는지 우리 팀에 권고**하는 방식을 그대로 재생산하게 하는 단일 지침.
> 대구: Reviewer는 **verdict(판정)**를 낸다. Council은 **counsel(권고)**를 낸다.
> red team 대비 **추가 의무 2가지**: (A) 모든 KPI/방법론에 **레퍼런스(근거)**를 단다 — 최근 논문·필드 표준. (B) 그 방법론을 **PoC로 증명하는 법**을 함께 준다.
> 방법론 짝: [`Reviewer.md`](Reviewer.md) · 마스터 인덱스: [`README.md`](README.md)

---

## 0. Council의 목표 (무엇을 산출하나)

Reviewer가 *“무엇이 틀렸나”*를 말했다면, Council은 세 가지를 묶어 답한다:
1. **개선안** — QA 정의·KPI를 `기존 → 제안`으로 **확정**(Reviewer의 Stage 2 권고를 근거로 구체화·결정).
2. **근거(레퍼런스)** — *왜* 이 KPI/방법이 맞는가. “최근 논문은 이렇게, 필드는 보통 이렇게” 형태로 출처를 단다.
3. **PoC 증명법** — *우리가* 이 방법론을 어떻게 작은 실험으로 증명하나. 도구·데이터·합격선·기간.

산출물은 “정답 강요”가 아니라 **팀이 채택을 결정할 수 있는 근거 있는 권고**다. 채택은 팀/Stage 2의 몫.

---

## 1. 입력 (권고 전 반드시 읽을 것)

- **해당 라운드 `round-NN/review/` 전체** — `report.md`(red team 결론), `QA-0X-*.md`(렌즈별 지적·Stage 2 권고), `_new-qa-candidates.md`. **Reviewer 의견을 빠짐없이 읽는 것이 1번 의무.**
- `context/qa/` 원본 QA·QAS, `context/overview.md`·`glossary.md`·`INDEX.md` (red team과 동일 기준선)
- `context/dp/` 설계결정 — 권고가 어느 DP로 실현되는지 연결
- 직전 라운드 `counsel.md` (있으면 — 권고 추세)

---

## 2. 3 seats (Reviewer 3렌즈와 동형의 합의체)

세 석(seat)이 각자 발의하고, 합의/소수의견을 표시한다. (Reviewer의 렌즈와 1:1 대응 — red가 깐 자리를 같은 전문성으로 blue가 메운다.)

| seat | 전문성 | 주로 메우는 약점 |
|---|---|---|
| **Seat 1** | Agentic Workflow 전문가 | eval/red-team 하네스, 비결정·일관성, 토큰 경제, HITL·runaway |
| **Seat 2** | 20년차 수석 아키텍트 | KPI 측정가능성·altitude, QA taxonomy, SLO/SLI 형식 |
| **Seat 3** | 대규모 Workflow Runner 인프라 아키텍트 | durable execution, 오토스케일·격리, 데이터 평면, 운영 PoC |

- 각 권고에 **발의 seat**과 **합의 여부**(consensus / Seat n dissent)를 표기.

---

## 3. 권고 3대 필수 요소

모든 QA 개선 권고는 아래 셋을 **반드시** 포함한다. 하나라도 빠지면 미완성.

1. **개선안** — `정의: 기존 → 제안`, `KPI: 기존 → 제안`. 미정 수치는 `◯`로 두되 산출 근거를 명시.
2. **근거(레퍼런스)** — 권장 KPI 패턴이 표준/연구에서 어떻게 쓰이는지 + 출처. §4 라이브러리를 먼저 활용하고, 부족하면 추가 검색.
3. **PoC 증명법** — §5 템플릿으로 1개 이상. “이 KPI가 달성/측정 가능함을 어떻게 보일 것인가”.

---

## 4. KPI 레퍼런스 라이브러리 (품질속성 유형별 — 라운드 무관 canon)

**품질속성 *유형*별로** 필드 표준 KPI 패턴 + 근거 + PoC 수단을 묶어둔다. 이 표는 **특정 라운드·특정 QA 번호에 묶이지 않는 재사용 자산**이다(매 라운드 재유도할 필요 없는 canon). 권고 작성 시 여기서 끌어 쓰고, 새 유형은 검색해 추가한다.

> ⚠️ **경계**: 이 라이브러리는 “어떤 KPI 패턴이 표준인가”만 담는다. *“이 라이브러리의 어느 항목을 QA-XX의 어느 약점에 적용할지”*는 그 라운드의 `counsel/`에서 결정한다(라운드 데이터를 이 스펙에 박지 않는다).

| 품질속성 유형 | 권장 KPI 패턴 (필드·논문) | 근거 (레퍼런스) | PoC 증명 수단 |
|---|---|---|---|
| **가용성·복구** (Availability) | **SLO(목표%) + error budget**; MTTR은 **durable execution**으로 “lease timeout + 재스케줄”로 분해, event sourcing으로 무손실 보장 | Google SRE(SLO/SLI·error budget) · Temporal durable execution(event history·exactly-once) | 노드 강제종료(chaos) 후 **작업 손실 0·재개 시간** 측정 |
| **지연** (Latency) | 평균 금지, **percentile SLI(p95/p99)**; 처리량은 throughput SLO | Google SRE · “p50/p95/p99” 가이드 | **k6/Locust 부하시험**으로 p95/p99·throughput 측정, SLO 통과 검증 |
| **확장성·처리량** (Scalability) | **Universal Scalability Law**(contention α·coherency β)로 scale 한계 모델링; **backlog 기반 오토스케일**(CPU 아님); scaling efficiency | USL(Gunther) · KEDA(큐 깊이 스케일) | 부하 N배↑ 시 **scaling efficiency** 측정 + KEDA 큐-깊이 스케일 시연 |
| **비용·토큰 효율** (Cost/Efficiency) | raw token 아닌 **$/task**; **cache read=정상가 10%(≈90%↓, 지연 85%↓)** 분리 집계 | Anthropic prompt caching·pricing | 캐시 On/Off **A/B로 $/task·TTFT** 비교 |
| **관측성** (Observability) | **OTel GenAI semconv**: `invoke_agent/chat/execute_tool` span + `gen_ai.usage.*` 토큰·model·finish_reason → **trace 완전성** | OpenTelemetry GenAI semantic conventions | 워크플로우 계측 후 **event-history/span 완전성 %** 측정 |
| **제어·안전** (Controllability/Safety) | **zero-trust 최소권한 + 고위험 액션 HITL 승인 + 별도 가드레일 모델 + injection 방어**; 위반율은 적대적 eval로 | Anthropic *Building Effective Agents* · 신뢰 에이전트 5원칙(human control 등) | **red-team eval 세트**로 허용범위 위반 통과율·injection 차단율 측정 |
| **일관성** (Consistency) | **pass^k**(k회 i.i.d. 시행의 일관성; pass@1 90%도 k=8엔 57%로 급락) — 캐시 우회 측정 | τ-bench (arXiv 2406.12045) pass^k | 고정 작업셋 **k회 반복 → pass^k** 산출 |
| **정확성** (Correctness) | **golden dataset(앵커 50~100) + LLM-as-judge(다수결, 인간 일치 75~90%)**; 100% 통과면 난이도 부족 신호 | golden-dataset·LLM-as-judge 베스트프랙티스 | golden set 구축 → judge **인간 일치도 검증** 후 정답률 측정 |
| **격리·멀티테넌시** (Isolation) | **bulkhead·큐별 동시성 제한·token-bucket rate limiter·fair queueing** | KEDA/큐 스케일 · SRE | 한 WF 폭주 주입 시 **타 WF 쿼터 침범·latency 증가** 측정 |

> 표의 “근거”는 패턴 정당화용이며, 우리 시스템의 실제 수치는 PoC로 직접 확정한다(레퍼런스 수치를 그대로 베끼지 않는다).

---

## 5. PoC 설계 가이드

각 권고는 **증명 가능**해야 한다. 아래 한 장 템플릿으로 적는다.

> ⚠️ **경계**: 이 절은 PoC 설계 *방법*(템플릿·아키타입)만 담는다(라운드 무관 canon). *구체 PoC* — 어느 QA의 어느 KPI를 무엇으로 증명하나 — 는 그 라운드 `counsel/_poc-plan.md`에 둔다.

```
### PoC-<번호>: <무엇을 증명하나>
- 가설(Hypothesis): "<KPI 제안>이 달성/측정 가능하다"
- 지표(Metric): 측정할 값 + 합격선(예: p95 ≤ ◯, pass^8 ≥ ◯)
- 셋업(Setup): 도구·데이터·환경 (k6 / chaos 주입 / golden set / 캐시 A/B …)
- 절차(Procedure): 단계 1~3
- 합격 기준(Exit): 무엇을 보면 “증명됨”인가
- 규모/기간(Scope): 최소 실험 크기, 예상 소요
- 리스크/한계: PoC가 못 보는 것 (silent cap 명시)
```

**PoC 아키타입** (§4 라이브러리의 9개 유형을 빠짐없이 덮는다):
- **부하시험** — k6/Locust로 p95/p99·throughput·scaling efficiency. CI/CD 통합·SLO 게이트 가능. → *지연·확장성*
- **장애주입(chaos)** — 노드/제공자 강제종료로 durable execution의 무손실·재개 검증. → *가용성·복구*
- **격리 주입(noisy-neighbor)** — 한 WF 폭주를 주입해 타 WF 쿼터 침범·latency 증가 측정. → *격리·멀티테넌시*
- **계측(instrumentation)** — OTel GenAI span 삽입 후 event-history/trace 완전성 측정. → *관측성*
- **eval 하네스** — golden set + LLM-as-judge(다수결)로 정확성; 적대적 세트로 제어·injection. → *정확성·제어·안전*
- **반복시행** — 동일 입력 k회로 pass^k(일관성), 캐시 우회 모드로 순수 추론 측정. → *일관성*
- **비용 A/B** — prompt caching On/Off로 $/task·TTFT 절감 실측. → *비용·효율*

> PoC는 **작게 시작**한다(앵커 50~100건, 부하 수십 VU 등). “전부 검증했다”는 착시를 막기 위해 PoC가 **못 본 부분을 반드시 log**한다.

---

## 6. 개별 QA 권고 파일 포맷 (`counsel/QA-0X-<slug>.md`)

```
# Counsel: QA-0X <속성명>

> refs-review: round-NN/review/QA-0X-*.md
> seats: 발의 <Seat n> · 합의 <consensus / Seat m dissent>
> stance: <채택 권장 / 조건부 / 신설 / 보류>

## Reviewer 지적 요약        — red team이 무엇을 깠나 (1~3줄, report.md/QA 파일 인용)
## 개선안 (정의·KPI 기존→제안)  — §3-1
## 근거 (레퍼런스)            — §3-2, §4 라이브러리 + 출처 URL
## PoC 증명법                — §3-3, §5 템플릿 1개 이상
## DP·발표 영향              — 어느 DP로 실현되나 / 번호 재정렬·발표 서사 영향
```

- cross-link은 ID 텍스트(`QA-03`, `DP-0002`, `NQA-A`)로. 레퍼런스는 각주/URL로.

---

## 7. 라운드 산출물 (blue team 몫은 `round-NN/counsel/`)

| `round-NN/counsel/` 파일 | 역할 |
|---|---|
| `counsel.md` | **이것만 읽으면 권고 전모를 아는 종합 보고서** — 채택 권고표·핵심 근거·PoC 로드맵·직전 대비 변화. **날짜 등 메타를 여기 기록**(공의회 최종 문서). |
| `README.md` | counsel 폴더 내비게이션 |
| `QA-0X-*.md` | QA별 개선 권고 (§6 포맷) |
| `_poc-plan.md` | PoC 통합 계획 — 우선순위·의존성·총 기간 |

상위 [`README.md`](README.md) 마스터 인덱스에 권고 추세(채택 상태)와 verdict 변화를 갱신.

---

## 8. 관리 규칙 / 실행 절차

**관리**: `review/`와 동일하게 **append-only**(라운드 보존). Council 권고가 `context/qa/`에 반영되면 QA `updated:` + `changelog.md` 델타, 다음 라운드 추세 갱신. QA·QAS 2자리 번호 = 우선순위(신설 채택 시 재번호 주의).

**절차 체크리스트**:
1. §1 입력 — **해당 라운드 review/ 전부** + context + dp 읽기.
2. QA별 §6 포맷으로 **개선안 → 근거(레퍼런스) → PoC** 3요소 작성. 3 seats 발의·합의 표기.
3. 신규 QA(NQA-*)는 채택 권고 + KPI 근거 + PoC까지.
4. `counsel.md`(종합·메타) + `_poc-plan.md` + counsel README 작성.
5. 마스터 인덱스 추세 갱신. 레퍼런스 수치 직접 베끼지 않았는지·PoC가 못 본 부분 log했는지 확인.

---

## 9. ★ 등급 척도(rubric) 캘리브레이션 — ATAM trade-off용

> Council이 **각 QA 주 KPI를 ★1~3 등급 척도로 캘리브레이션**하는 방법(라운드 형태 무관 canon). 산출물 = 각 QA 파일의 `## 등급 척도` 섹션(`context/qa/`). 용도 = ATAM에서 **동일 조건 두 아키텍처 대안을 비교해 ★ 많은 안 채택**(★ = 설계 대안이 이 QA를 얼마나 잘 만족시키는가의 절대 수치 구간).
>
> **급간 클레임은 어느 경로로든 들어온다**: KPI 급간 경계에 대한 지적(예: "이 ★★★는 필드상 도달 불가", "하한이 비현실적이라 ★★☆가 사문화")은 **red-team Reviewer가 다른 KPI 비평과 함께 제기**할 수도, **팀이 직접 요청**할 수도 있다. 경로와 무관하게 Council은 아래 9.1 규칙으로 필드 근거+PoC margin 캘리브레이션을 산출한다. (round-03은 팀이 reviewer 자리를 대신한 reviewer-less 형태로 돈 1례일 뿐 — 이 방법이 reviewer-less 전용은 아니다.)

### 9.1 5대 규칙

1. **하한 = ★☆☆ 진입선**. KPI에 적힌 합격선(하한)이 ★☆☆ 진입선, 미만은 불합격(별 없음). ★★☆·★★★는 위로 올라가되 **실제 필드 도달 범위**로 캘리브레이션 — 웹 레퍼런스(필드 벤치마크·논문·업계 표준) 필수, 수치 복제 금지.
2. **비현실 하한 flag + (B) 재배치**. 하한이 필드 기준 비현실적으로 높아 ★★★가 죽은 등급이 되면, 캘리브레이션 노트에만 적지 말고 **표 급간 자체를 현실 대역으로 재배치**한다(KPI 하한 사실상 보정). 보정 사실을 노트에 명시하고, 하한 보정은 `open-issues.md` 등록(KPI 정의 섹션과 정합 확인).
3. **현실 급간에 PoC margin**. 이론 천장(웹 근거)에 급간을 빡빡하게 붙이지 말 것 — 우리 PoC는 가상 설계·제한 구현이라 이론 완성도 미달이 예상되므로 **이론치보다 다소 낮게/넓게** 잡아 margin을 둔다. 노트에 `이론 근거 X + PoC 미완성 대비 margin Y` 형태로 명시.
4. **2-index → main + 조건**. KPI가 2개 이상 지표(A·B)면 **PoC 측정 가능한 쪽을 main 급간 축**에, 나머지는 표 밖 `조건:`/`측정 조건:` 줄로 고정("B = xxx 고정 하에 A를 급간"). seat dissent는 이 구조로 흡수해 consensus 전환.
5. **보조지표 0건 가드 → Constraint 분기**. binary/대조형 헤드라인은 별점 급간이 무의미 → pass/fail 게이트로 두고 별점은 **gradable한 보조지표**에 매긴다. 단 그 보조지표가 **또 0건/위반 절대형**(1·2·3건을 허용할 게 아님)이면 급간 원천 불가 → QA 아니라 **Constraint(C)로 보낸다**. gradable proxy가 있을 때만 QA로 성립(없으면 QA 전체가 Constraint 후보). 가짜 급간 날조 금지.

### 9.2 섹션 포맷 (`context/qa/QA-0X-*.md`에 추가)

```
## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건 설계 대안의 본 QA 만족도를 ★1~3 비교(별 많은 안 채택). KPI 합격선(하한)=★☆☆ 진입선, ★★☆/★★★는 필드 현실 도달 범위+PoC margin. 하한 미만 불합격. 예시값이며 경계는 PoC로 확정.

**조건 (2-index일 때):** `조건: <B 고정값>` — <왜 고정하나>

| 등급 | 구간 — 주 KPI(main 축): <KPI명> | 필드 근거 (경계 이유 + URL) |
|---|---|---|
| ★★★ (상) | <범위> | <필드 최상위 − margin + 출처> |
| ★★☆ (중) | <범위> | <필드 일반 우수 + 출처> |
| ★☆☆ (하) | <범위 — 합격 최소선=(보정된) KPI 하한> | <합격 진입 근거> |
| 불합격 | <하한 미만 / 게이트 위반> | — |

> **캘리브레이션 노트**: <하한 현실성 판정 + (재배치 시) 이론 근거 X + PoC margin Y + 신뢰도 상한(커버리지 등) silent cap>
> **seats**: 발의 <Seat n> · <consensus / Seat m dissent>
```

- binary/대조형 헤드라인은 표 위에 한 줄: "헤드라인 `<...>`은 0건 절대형(Constraint 성격) → 별점은 보조지표 `<X>`로 급간화 (gradable이라 QA 성립)".

### 9.3 운영

- seats(§2)·레퍼런스 라이브러리(§4)·PoC margin 정신은 표준 council과 공유. **1~2개 샘플 먼저 확인 후 fan-out**(프로젝트 규칙).
- 반영 시 QA `updated:` + `changelog.md` 델타. 재배치로 KPI 하한을 보정하면 KPI 정의 섹션·`open-issues.md`와 정합 확인.

---

## 레퍼런스 (라이브러리 출처)

- Google SRE — SLO/SLI·error budget: https://sre.google/sre-book/service-level-objectives/ , https://sre.google/workbook/implementing-slos/
- Latency percentiles p50/p95/p99: https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view
- Temporal durable execution: https://temporal.io/blog/what-is-durable-execution , https://docs.temporal.io/temporal
- Universal Scalability Law: https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/
- KEDA 이벤트 기반 오토스케일: https://keda.sh/
- Anthropic prompt caching·pricing: https://platform.claude.com/docs/en/build-with-claude/prompt-caching , https://platform.claude.com/docs/en/about-claude/pricing
- OpenTelemetry GenAI semantic conventions(agent spans): https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/ , https://opentelemetry.io/blog/2025/ai-agent-observability/
- Anthropic *Building Effective Agents* / 신뢰 에이전트 프레임워크: https://www.anthropic.com/engineering/building-effective-agents , https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents
- τ-bench (pass^k): https://arxiv.org/abs/2406.12045 , https://sierra.ai/blog/tau-bench-shaping-development-evaluation-agents
- LLM-as-judge·golden dataset: https://www.comet.com/site/blog/llm-as-a-judge/ , https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/ , https://montecarlo.ai/blog-llm-as-judge/
- k6 부하시험(SLO 검증·CI/CD): https://k6.io/
