# Counsel: QA-11 Reliability — Agent 결과 일관성

> refs-review: round-04/review/QA-11-reliability-agent-consistency.md
> seats: 발의 Seat 1 (비결정·일관성·eval 하네스) · 합의 consensus (Seat 2 main 축 이중화·Seat 3 표본/runner 동의)
> stance: **조건부 채택 — ★★★ 하향(≥60%→≥45%) + 보조 별점 축(②-1) 병기 + apples/k 보간 silent cap**

## Reviewer 지적 요약
round-04 review의 **가장 날카로운 단일 지적**: ★★★ pass^5 ≥60%는 frontier pass^1 ~80% → 독립시행 0.8^5≈33%인데, voting이 33→60%를 끌어올린다는 근거가 인용에 없어 **사문화(아무도 못 받음) 의심**(권고 1·Med). 부수: ★☆☆ 25% 하한이 GPT-4o **pass^8** 값이라 k=5 main 축과 k 혼동(권고 2), main 별점 축(pass^5)이 [발표 서사]라 실측 불가 → ②-1/H_norm을 보조 별점 축 병기(권고 4·C3), τ-bench retail vs SDK 빌드 apples-to-apples(권고 5·C1), §측정 50행 구값 `pass^k 70%` 잔존(권고 3).

## 개선안 (정의·KPI 기존→제안)

### (A) ★★★ pass^5 하향 — **가장 중요**
- **정의**: 변경 없음(유효-결정 안정성, pass^5 main 축 유지).
- **KPI ★ 급간 기존 → 제안**:
  - 기존: `★★★ pass^5 ≥60% (frontier+voting 가정 최상위)`
  - 제안: **`★★★ pass^5 ≥45%`** — 근거는 아래 §근거에서 **voting이 pass^k를 끌어올린다는 가정이 잘못된 메커니즘 이해임을 1차 출처로 확정**했기 때문. ★★☆는 `35%≤pass^5<45%`로 조정(상한을 60→45로 내렸으므로 중급 대역도 동반 압축), ★☆☆는 `25%≤pass^5<35%` 유지(하한 25는 보간 주석 추가).
  - **단조성·변별력 유지**: 25 / 35 / 45 비등간격(10/10/∞)으로 곡선 급감 반영. ★★★를 0.8^5≈33%(이론 상한 근방)보다 **약간 위(45%)**에 둬 "frontier + 약한 상관성(temp=0·결정성 레버)로 도달 가능한 상위"로 재정의 — 사문화도 안 되고(45%는 frontier가 결정성 강화 시 도달 가능 대역), 물러지지도 않음(GPT-4o pass^4 ~37%보다 위).

### (B) 보조 별점 축 병기 (C3 — main 축 실측불가 보강)
- **기존**: ②-1(룰 게이트 통과율)·H_norm은 **동반 게이트(pass/fail)**로만 소비.
- **제안**: **②-1(대리 유효-결정률)을 보조 별점 축으로 병기**해 "모델 추정(pass^5, main) + 실측 가능(②-1, 보조)" **이중 축**. ②-1은 우리 조건에서 룰 체커로 실제 산출 가능(golden 불요). 보조 ★: `②-1 ≥99% = ★★★ / 97~99% = ★★☆ / 95~97% = ★☆☆`. H_norm은 게이트 유지(≤0.2). ATAM 두 설계 비교 시 pass^5가 동률/미실측이면 ②-1로 변별.

### (C) k 보간 silent cap (권고 2)
- ★☆☆ 25% 하한 옆에 `(25%는 GPT-4o pass^8 값의 보수적 차용 — k=5 환산 시 GPT-4o pass^5≈33%라 하한에 여유. pass^5 직접 보고치 확보 시 교체)` 명시.

### (D) §측정 50행 구값 정정 (권고 3·Low)
- `pass^k 70%` → `pass^5 25/35/45%`로 정정(보정 후 표와 일치).

## 근거 (레퍼런스 + 검증 결과)

**C4 웹 검증 — τ-bench 인용은 실재·정확**:
- arXiv 2406.12045 **실재**, GPT-4o τ-retail **pass^1 61.2% / pass^4 ~37% / pass^8 ~25%** 정확히 확인. frontier pass^1 ~80% crossing(retail) 확인. ([τ-bench arXiv](https://arxiv.org/abs/2406.12045) · [Sierra τ-bench](https://sierra.ai/blog/benchmarking-ai-agents))

**핵심 — self-consistency voting의 pass^k 효과(★★★ 재판정의 결정적 근거)**:
- **voting은 pass^k를 직접 끌어올리지 못한다.** self-consistency/majority voting은 **k개 샘플을 다수결로 하나의 답으로 합쳐 그 단일 답의 정확도를 높이는** 기법이다(GSM8K +17.9%, MATH 등에서 single-path 대비 향상). 그러나 pass^k는 **k회 독립 시행이 *전부* 성공할 확률**(all-k-succeed)이다 — voting은 "1개의 견고한 답"을 만들 뿐 "k개의 일관된 시행"을 만들지 않는다. 둘은 서로 다른 지표다. ([Self-Consistency](https://www.emergentmind.com/topics/self-consistency-sampling) · [pass@k vs voting](https://leehanchung.github.io/blogs/2025/09/08/pass-at-k/) · [Certified Self-Consistency arXiv 2510.17472](https://arxiv.org/pdf/2510.17472))
- 따라서 "voting이 0.8^5≈33%를 60%로 끌어올린다"는 round-03 정당화는 **메커니즘 오해**다. 실제로 voting을 **시스템에 내장**하면(에이전트가 매 시행 내부적으로 k-샘플 투표) 각 시행의 성공확률 p 자체가 올라가 pass^5=p^5도 오르지만, frontier p가 voting으로 80%→90%대가 되어도 pass^5는 0.9^5≈59%로 **간신히 60% 부근**이다(완전 독립 가정). 우리 조건(가상 설계·실측 불가)에서 frontier+voting 내장+상관성을 모두 최상위로 가정해야 60%인데, 이는 **이론 상한에 margin 없이 붙인 낙관** → 규칙3 위배.
- **결론**: ★★★를 **45%**로 두면 (a) "frontier(p≈80~85%) + 결정성 레버(temp=0·fixed seed) → 시행 간 약한 양의 상관"으로 pass^5가 독립 가정(33%)보다 위로 끌리는 현실(상관 시행은 pass^k가 독립보다 항상 높음)을 **margin 있게** 반영하고, (b) GPT-4o pass^4 ~37%보다 위라 변별력 유지. 60%는 못 대고 45%는 근거로 댄다.

**C1 apples-to-apples 검증**:
- τ-bench는 **retail/airline 고객응대 멀티턴 도구사용**, 우리는 **SDK 빌드 파이프라인(IR→Optimizer→Quantizer→Compiler)**. "유효 결정"의 난이도 프로파일이 달라 τ-bench 대역 차용은 **가정**임을 silent cap 명문화.

## PoC 증명법

### PoC-11A: ★★★ 45% 재판정 — voting의 pass^k 효과 실측 (반복시행 아키타입)
- 가설: "★★★ pass^5 경계는 60%가 아니라 45% 부근이 근거 있는 상한이다 — voting은 pass^k를 60%로 끌어올리지 못한다."
- 지표: 단일-시행 성공확률 p와 voting 내장 시 p', 그리고 pass^5=측정 일관율. 합격선: voting 내장 설계의 pass^5가 45% 도달 가능·60%는 미도달.
- 셋업: mock 에이전트(고정 p를 주는 stochastic 결정기) + self-consistency 래퍼(내부 k'=3·5 투표). 캐시 우회 모드. control plane이 동일 task를 5 worker fan-out, model/prompt/seed 핀닝, durable replay.
- 절차: ① voting 없는 baseline에서 p별 pass^5 곡선(=p^5 근사) 측정 → 33% 부근 확인. ② voting 내장 시 p'→pass^5 상승폭 측정. ③ 45% vs 60% 도달 여부 판정.
- 합격 기준: voting 내장으로도 pass^5가 60%엔 frontier 가정에서만 닿고 45%엔 일반 우수에서 닿으면 "★★★=45%가 근거 있음" 증명.
- 규모/기간: N=20~50 task × 5회 × {voting on/off}, ~0.5일(mock).
- 리스크/한계(silent cap): pass^5 **절대값 실측은 실제 에이전트 필요**(미실행) — PoC는 "voting의 pass^k 메커니즘과 경계 위치"를 모델로 보일 뿐.

### PoC-11B: 보조 별점 축 ②-1 산출 (반복시행 — 우리 조건 산출 가능)
- 가설: "②-1(룰 게이트 통과율)은 우리 조건에서 실측되어 보조 별점 축이 된다."
- 지표: 재실행 k=5 결과의 hard-invalid 차단율. 합격선: ②-1 ≥95%(★☆☆)~99%(★★★) 변별.
- 셋업: 룰 체커(C-제약·allowlist·스키마 위반) + 라벨 매핑 컴포넌트(QA-06 차단율·H_norm과 공유 자산).
- 합격 기준: 동일 룰셋에서 두 mock 설계가 서로 다른 ②-1을 내 ATAM 변별 가능.
- 규모/기간: ②-1은 결정론적 룰이라 ~0.3일.

## DP·발표 영향
- **DP-0005(공유 캐시)**가 **캐시 우회 모드**를 명시 제공하는지 DP 역검토(★ main 축 측정 전제). voting 내장은 신규 tactic 후보(DP 디스커션).
- 발표 서사: "★★★를 60→45%로 **정직하게 하향** — voting이 pass^k를 끌어올린다는 흔한 오해를 1차 출처로 반증하고, 실측 가능한 ②-1을 보조 축으로 병기해 ATAM 변별력을 살림"은 **방법론 성숙도를 보여주는 강한 서사**.
- 번호·우선순위 영향 없음(importance L 유지). OI-7(②-2↔QA-07 golden) 잔존 트래킹.
