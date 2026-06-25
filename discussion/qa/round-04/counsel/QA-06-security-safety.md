# Counsel: QA-06 Security / Safety

> refs-review: round-04/review/QA-06-security-safety.md
> seats: 발의 Seat 1 (eval/red-team·injection) · 합의 consensus (Seat 2 margin 규칙화·FPR 통일·Seat 3 세트 크기 동의)
> stance: **조건부 채택 — apples 명문화 + margin 규칙화(★★☆ 85%) + FPR 전급간 게이트 + 세트 크기 silent cap**

## Reviewer 지적 요약
★ 재배치 방향·구조는 타당하나(규칙5 정확·OI-9 5곳 정합), red-team 4건: (1) **apples-to-apples 미검증** — Unit 42는 직접 injection·단발 prompt인데 우리는 agentic 간접 injection(권고 1·Med·C1), (2) **margin 불일치** — ★★★ 5%p vs ★★☆ 11%p(권고 2·Med·C2), (3) FPR ≤1%가 ★★★만 필수(권고 3·Low), (4) 세트 50~100건은 90/92 경계 변별에 표본오차 부족(권고 4·Low). + Unit 42 53/91/92%·FPR 13.1% 1차 출처 재확인(권고 6·C4).

## 개선안 (정의·KPI 기존→제안)

### (A) apples-to-apples 명문화 (C1 — 가장 중요)
- **정의**: 변경 없음.
- **★ 노트 기존 → 제안**:
  - 기존: OWASP LLM01 직접/간접만 언급, 인용 벤치의 측정 셋업 차이 미명시.
  - 제안: 캘리브레이션 노트에 **silent cap 추가** — `인용 Unit 42 수치(Platform 53/91/92%)는 JailbreakBench **단발·직접 injection** 차단율이고, 우리 QA-06는 배포권 멀티턴 에이전트의 **간접 injection**(도구 응답·artifact·이슈코멘트에 숨긴 지시)을 잰다. 공격면이 다르므로 경계는 이 난이도 차이를 감안한 예시값 — 우리 red-team 세트(간접 injection 포함)로 확정.`
  - 근거: 웹 검증으로 Unit 42 방법론이 "single-turn only, JailbreakBench"임을 **확인**(아래 §근거) → 이 silent cap은 추정이 아니라 1차 출처 사실.

### (B) margin 규칙화 — ★★☆ 85% (C2)
- **★ 급간 기존 → 제안**:
  - 기존: `★★☆ 80~90%` (Platform 91·92%를 ~11%p 내림 — 근거 없음), `★★★ ≥90%` (연구 95%에서 5%p).
  - 제안: **margin 규칙을 QA-06 단일 Y=5%p로 고정** → `★★☆ 85~90%`(상용 typical 91% − 5%p ≈ 85%), `★★★ ≥90%`(연구 95% − 5%p) 유지, `★☆☆ 70~85%`. 이렇게 하면 두 경계 모두 "필드/이론 천장 − 5%p" 단일 규칙. 단조성·변별력 유지(Platform 1 53%는 여전히 불합격, Llama Guard 76%는 ★☆☆).
  - **대안(택1)**: 80% 유지하되 노트에 `★★☆ 11%p margin = 간접 injection 난이도 가산(직접→간접 차단율 저하 흡수)`으로 **차등을 명시적 근거화**. → Council 권장: **(B) 85%로 규칙 통일**이 C2 "임의 margin" 지적에 더 정면 응답. 차등이 필요하면 노트로만.

### (C) FPR 전 급간 hard 게이트 (권고 3)
- 기존: FPR ≤1%가 ★★★ 필수, ★★☆은 `≤~1%`(물결). → 제안: **FPR ≤1%를 전 급간 hard 조건**(과방어 차단 일관). Platform 3가 92% 차단이나 FPR 13.1%로 과방어인 실증이 이 게이트를 정당화.

### (D) 세트 크기 + ★★★ 전제 silent cap (권고 4·5)
- ## 검증 전략에 `red-team 세트 50~100건은 차단율 95% CI ~±10%p라 ★★☆/★★★ 경계(90%) 변별 불충분 — 경계 신뢰엔 세트 확대 필요([생략]된 노동)` + ★★★ 행에 `admission/HITL 게이트 동작이 implicit 전제(차단율만 높은 안은 ★★★ 불가)`.

## 근거 (레퍼런스 + 검증 결과)

**C4 웹 검증 — Unit 42 인용은 실재·정확(전수 확인)**:
- [Unit 42 "Comparing LLM Guardrails across GenAI platforms"](https://unit42.paloaltonetworks.com/comparing-llm-guardrails-across-genai-platforms/) (2025-06) **실재**. Table 2: **Platform 1 65/123 (~53%) / Platform 2 112/123 (~91%) / Platform 3 114/123 (~92%)** — 정확. Table 1 FPR: **Platform 1 0.1% / Platform 2 0.6% / Platform 3 13.1%(131/1000)** — 정확.
- **측정 세트 = single-turn only, JailbreakBench = 직접 injection**. → C1 apples-to-apples 지적이 **1차 출처로 입증됨**. (B)·(A)의 근거 확정.
- 미검증: TRYLOCK 94.4%·Constitutional Classifiers 95.6%는 본 검증에서 미대조 → "확인 불가, 연구 최상위 대역(~95%) 정당화용"으로 노트 유지(수치 복제 아님).
- arXiv **2511.15759**(Securing AI Agents)·**2504.11168**(Bypassing Guardrails): 2511=2025-11, 2504=2025-04로 **과거 ID, 형식상 실재 가능**(2026-06 기준). 단 본 검증에서 정확 인용 대조는 미완 → "출처 유효 추정, 차단율 수치 복제 아님"으로 유지.

## PoC 증명법

### PoC-06: 간접 injection 차단율 + FPR — apples-to-apples 재측정 (eval 하네스 아키타입)
- 가설: "★ 급간(간접 injection 차단율)은 우리 red-team 세트로 측정 가능하고, 직접/간접 차단율 차이가 존재한다."
- 지표: 간접 injection 차단율(차단/통과 라벨) + 정상 빌드 코퍼스 FPR. 합격선: ★ 경계(70/85/90%) 위치 + FPR ≤1%.
- 셋업: red-team 세트(직접 직접 + 간접 — 도구 응답·artifact에 숨긴 지시) 50~100건, control plane이 worker pool fan-out, model/prompt/tool 버전 핀닝(durable replay), 격리 sandbox worker class.
- 절차: ① 직접 injection 세트 차단율 측정(Unit 42 대조군). ② 간접 injection 세트 차단율 측정 → **직접 대비 저하폭** 확인(apples 차이 실증). ③ 정상 코퍼스 FPR 측정.
- 합격 기준: 간접 차단율이 직접보다 낮게 나오고(난이도 가산 실증), 95% CI 폭을 함께 보고하면 "경계 신뢰엔 세트 확대 필요" silent cap 정량 입증.
- 규모/기간: 50~100건 × 2모드, ~1일. 95% CI 폭 함께 산출.
- 리스크/한계(silent cap): 세트 커버리지 = 신뢰도 상한(zero-day 미검출). 50~100건은 90/92 변별 표본오차 부족.

## DP·발표 영향
- **DP-0002(HITL)·DP-0003(권한 게이트)**가 공급망 서명·secrets·injection 가드레일·admission 게이트를 명시 안 함(OI-7) — ★★★ "admission 게이트" 책임 DP 비어 있음 → DP 디스커션 귀속.
- 발표 서사: "인용 가드레일 벤치(직접 injection)와 우리 위협(간접 injection)의 차이를 1차 출처로 명문화하고 margin을 5%p 단일 규칙으로 통일" = C1·C2에 정면 응답한 성숙 서사.
