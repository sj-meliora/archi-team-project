# Reviewer — QA 리뷰 재생산 스펙

> 목적: reviewer agent가 **이 디렉터리에서 확립한 QA 리뷰 방식을 그대로 재생산**할 수 있게 하는 단일 지침.
> 적용 대상: `context/qa/`의 QA(품질속성)·QAS(시나리오)·KPI.
> 한 번의 실행 = **1 라운드** = QA 전체 스냅샷 판정 + report.

---

## 0. 리뷰의 목표 (무엇을 판정하나)

각 QA에 대해 두 질문에 답한다:
1. **이 QA가 품질속성으로서 말이 되는가(sound)?** — 아니면 더 sound한 QA로 교정.
2. **KPI가 make sense 하고 현실적인가?** — 아니면 측정 가능·달성 가능한 KPI로 교정.

산출물은 “정답”이 아니라 **Stage 2(실제 QA 수정)의 근거**다. 권고는 `기존 → 제안` 형태로 적어 적용 가능하게 한다.

---

## 3. 평가 렌즈 (3종 — 반드시 셋 다 적용)

리뷰어는 아래 세 전문가 페르소나를 **각각 독립 섹션**으로 적용한다. 한 렌즈가 다른 렌즈를 대신하지 않는다.

### 렌즈 1 — Agentic Workflow 최상위 전문가 (Anthropic/OpenAI 급)
LLM 에이전트의 정의적 특성에서 QA를 본다. 점검 축:
- rate limit(TPM·RPM)·토큰 경제·prompt caching·context 관리
- 비결정성·환각·self-consistency·eval/red-team 하네스
- 외부 제공자 의존(장애·버전)·도구 사용·HITL·runaway loop
- “일반 분산시스템 어휘”로 뭉개진 곳에 **agentic 고유 리스크**가 빠졌는지

### 렌즈 2 — 20년차 수석 아키텍트 (QA 완성도)
QA/QAS 방법론의 형식적 완성도를 본다. 점검 축:
- **Sound**: 단일 관심사인가? 다른 QA와 경계가 분명한가(중복 없음)?
- **Measurable**: KPI에 구체 임계값·측정 방법이 있는가? (placeholder·“최대화” 금지)
- **Realistic**: 그 수치가 달성 가능한가?
- **Consistent**: 정의 ↔ KPI ↔ QAS가 같은 것을 가리키는가?
- altitude(관심사 위치)가 맞는가 — top-line인가 하위 tactic인가?

### 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트
워크플로우를 수행하는 **runner를 어떻게 구성·확장·효율화**하는지의 관점. “이 QA를 runner 아키텍처로 어떻게 실현/측정하나?”를 묻는다. 점검 축:
- **runner 구성**: control plane(스케줄러)/data plane(worker) 분리, pull 기반 worker pool, 영속 큐, durable execution(이벤트 소싱·체크포인트·replay)
- **확장**: backlog 기반 오토스케일(CPU 아님), worker class 분리, bin-packing, admission control/backpressure
- **자원 효율**: warm pool·affinity, 배칭·co-location, spot/preemptible, 가동률 vs headroom
- **격리/제어**: 큐별 쿼터·동시성 제한, token-bucket rate limiter, fair queueing, circuit breaker, activity timeout/retry cap, 협조적 취소+hard-kill
- **데이터 평면**: artifact 참조 전달·data locality·공유 스토어
- 각 렌즈3 말미에 **runner 측 KPI**(측정 가능한 형태)를 제시한다

> 렌즈3는 추상 비판이 아니라 **“어떻게 만들/측정할 것인가”의 구현적 답**을 주는 자리다. 가능하면 구체 메커니즘(Temporal류 durable execution, token-bucket, blue-green 등)으로 환원한다.

---

## 1. 입력 (리뷰 전 반드시 읽을 것)

- `context/qa/QA-*.md`, `context/qa/QAS-*.md` (대상 전체)
- `context/overview.md` (시스템 정의·Pain Point — KPI 현실성 판단의 기준선)
- `context/glossary.md` (약어·QA 번호 매핑)
- `context/INDEX.md` (ID 체계·DP 목록)
- 직전 라운드 `round-NN/review/report.md` (있으면 — 추세 비교용)

## 2. 판정 rubric

| 축 | 질문 | 합격 조건 |
|---|---|---|
| **Sound** | 진짜 품질속성인가? 경계가 분명한가? | 단일 관심사 · 중복/혼동 없음 · 시스템 가치와 직결 |
| **Measurable** | KPI가 테스트 가능한가? | 구체 임계값 · 측정 방법 정의 · placeholder/“최대화” 금지 |
| **Realistic** | agentic·on-device 현실에서 달성 가능한가? | LLM 비결정·rate limit·토큰 경제·외부 의존 반영 |
| **Consistent** | QA ↔ QAS ↔ 정의가 일치하는가? | 같은 KPI, 정의와 KPI가 같은 것을 측정 |

**판정 표기**: ◎ 우수 · ○ 타당 · △ 부분 결함 · ✕ 재설계 필요
**verdict 형식**: `Sound <기호> / KPI <기호>` + severity(High/Med/Low)
**severity 기준**: 측정불가·정의↔KPI 불일치 = High / 누락·중복·현실성 결함 = Med / 경미·보강 = Low

---

## 4. 개별 QA 리뷰 파일 포맷 (`QA-0X-<slug>.md`)

```
# Review: QA-0X <속성명>

> source: context/qa/QA-0X-*.md + QAS-0X-*.md
> verdict: **Sound <기호> / KPI <기호>** — <한 줄 판정> · severity **<High/Med/Low>**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약          — 정의 · KPI · QAS 핵심을 3~4줄로
## 렌즈 1 — Agentic Workflow 전문가 관점
## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)
## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점   (말미에 runner 측 KPI)
## 판정               — 4축(sound/측정가능/현실성/일치) 표 + 근거
## Stage 2 권고       — 정의 재서술 · KPI `기존 → 제안` · DP 역검토
```

- cross-link은 ID 텍스트(`QA-03`, `DP-0002`, `QAS-05`)로 적어 grep 역참조되게.
- KPI 제안은 반드시 측정 가능 형태로. 미정 수치는 `◯`로 두고 산출 근거(예: overview의 140모델)를 메모.

---

## 5. 교차(cross-cutting) 분석

개별 QA를 넘어 **세트 전체의 구조적 문제**를 `C1, C2, …`로 번호 매겨 정리한다. 최소 점검:
- 측정 불가 KPI 군집 / 정의↔KPI 불일치 / QA 경계 중복(taxonomy) / agentic 고유 리스크 누락 / **누락된 1급 QA 후보**(→ `_new-qa-candidates.md`).

신규 QA 권고는 `NQA-A, B, …`로 매기고 `_new-qa-candidates.md`에 제안 정의 + KPI 초안 + 채택 영향까지 적는다. (성격은 **권고**이며 채택은 Stage 2 결정.)

---

## 6. 라운드 산출물 (red team 몫은 `round-NN/review/`)

라운드 폴더는 `round-NN/` (날짜 없음 — 메타는 보고서에 기록). red team 산출물은 그 아래 `review/`에 둔다(`counsel/`은 blue team [`Council.md`](Council.md) 몫).

| `round-NN/review/` 파일 | 역할 |
|---|---|
| `report.md` | **이것만 읽으면 라운드 전체를 아는 결론 요약** — 판정표·교차발견·우선순위·신규QA·직전 라운드 대비 변화. **날짜 등 메타를 여기 기록.** |
| `README.md` | 폴더 내비게이션(파일 목록·읽기순서·링크). 분석 내용은 두지 않음 |
| `QA-0X-*.md` | QA별 3렌즈 상세 (위 §4 포맷) |
| `_new-qa-candidates.md` | 신규 QA 권고(NQA-*) |

상위 `discussion/qa/README.md`(마스터 인덱스)에 라운드 1줄 + verdict 추세표를 갱신한다.

---

## 7. 관리 규칙 (discussion/qa/)

- **append-only**: 지난 라운드는 수정 금지(오타·링크만 예외). 개선은 새 라운드로.
- **순환**: 리뷰(라운드 N) → Stage 2에서 `context/qa/` 개선 → 라운드 N+1로 검증.
- **추적성**: 권고가 `context/qa/`에 반영되면 해당 QA `updated:` + `changelog.md` 델타로 남기고, 다음 라운드 report에 verdict 변화를 기록.
- **번호 주의**: QA·QAS는 2자리이고 번호 = 발표 우선순위(재정렬 가능). 신규 QA 채택 시 재번호 영향 검토.

---

## 8. 리뷰 실행 절차 (체크리스트)

1. §1 입력 전부 읽기 (overview·glossary 포함 — 현실성 기준선).
2. QA별로 §4 포맷에 따라 **3렌즈 모두** 작성 → 4축 판정 → Stage 2 권고.
3. 세트 전체로 §5 교차분석(C*) + 신규 QA(NQA-*) 도출.
4. `report.md` 작성(§6) — 판정표·교차발견·우선순위·신규QA·직전 대비 변화.
5. `README.md`(내비) + 마스터 인덱스 추세표 갱신.
6. append-only·추적성(§7) 준수 확인.
