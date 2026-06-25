# Review: QA-05 Efficiency

> source: context/qa/QA-05-efficiency.md + QAS-05-efficiency-token-budget.md
> verdict: **Sound ◎ / KPI ○** — 기존 tier(4/6/8k) 정식화는 캐싱 역페널티 회피가 모범. 잔여는 단일 출처(Requesty)에 전 경계가 매달리고 적중률 ≥85% 조건이 ★ 변별을 사실상 가린다 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(신규 토큰 4/6/8k tier 정식화) 검증 무게중심.

## 원문 요약
- 정의: 모델 1건 총비용(토큰+compute) 최소화, 캐싱 페널티 없는 측정(신규/캐시 분리).
- 헤드라인: `신규 토큰 ≤6k 일반/≤8k 복합 + 캐시 분리집계`.
- ★ 급간: main = 작업당 신규 토큰(cache-miss+output, 역방향). ★★★ ≤4k / ★★☆ 4~6k / ★☆☆ 6~8k / 불합격 >8k 또는 적중률<85%. 조건: 캐시 적중률 ≥85% 고정.

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
"raw 토큰이 캐싱을 역페널티" → 신규/캐시 분리 측정은 prompt caching(cache read ≈ base 10%)을 정확히 반영한 agentic 토큰 경제 통찰. 그러나 **★ 경계 전부가 Requesty "Coding Agent Economy" 단일 출처**(92% cache, 86% platform avg)에서 나왔다 — 단일 벤치 과의존(C군집). 또 apples-to-apples: Requesty는 **코딩 에이전트** 데이터인데 우리는 **SDK 빌드 파이프라인 노드**(빌드로그·config diff)다. 코딩 에이전트 적중률 86~92%가 우리 파이프라인 노드에도 성립한다는 보장이 없다(워크로드 유사성 의존, 이미 silent cap). → round-05 Council이 코딩 외 도메인 적중률 보강.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙4·5 적용**: 분리집계를 게이트로, gradable한 신규 토큰에 별점, 적중률 ≥85%를 조건으로. 정확. ○.
- **급간 reasonableness**: ★★★ ≤4k · ★★☆ 4~6k · ★☆☆ 6~8k. **등간격(2k 폭)** — 작업 난이도(단일/일반/복합)에 매핑한 등간격이라 자의적이지 않으나, "왜 2k 간격인가"의 근거가 표에 약함(Requesty 92%/86% 두 점에서 4k/6k 경계 파생, 8k는 "대입력 복합 margin"). margin이 명시적이지 않음(C2 군집).
- **적중률 조건이 ★ 변별을 가림(가장 날카로움)**: `조건: 적중률 ≥85% 고정`인데, 신규 토큰(cache-miss)은 **적중률의 직접 함수**다 — 적중률을 85%로 고정하면 신규 토큰 차이의 상당 부분이 사라져, ★ 급간이 잴 수 있는 건 "동일 적중률에서 출력 토큰·cache-miss 잔차"뿐. 즉 조건이 main 축의 변별 폭을 스스로 좁힌다. 정합성 자체는 옳으나(공정 비교 의도) ★ 변별력은 약화. → silent cap 권고.
- **하한 보정 정합(OI-9)**: §측정(35~38행 ≤6k/≤8k+tier), 등급표, 변경이력, counsel(6k→8k tier), QAS-05(`≤6k AND tier ★★★≤4k/★★☆4~6k/★☆☆6~8k`) **일치**. 재배치 불요(8k 현실적). glossary 없음. **OI-9 통과.**
- **변별력**: 하한 8k 비현실 아님 → 사문화 없음. ○.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
신규 토큰 = caching A/B(On/Off)로 측정. 적중률은 warm pool·prompt 구조에 의존 → runner가 캐시 워밍·affinity로 적중률을 올려야 ★ 도달. mock 파이프라인 적중률이 실운영보다 낮을 수 있어(이미 노트) ★ 경계 보수적. runner KPI: `적중률 분포가 워크로드 다양성에 robust(다양성↑ 시 적중률↓·신규 토큰↑ 곡선)` — 조건(85%) 충족 가능 여부가 ★ 전제.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | per-request 효율 단일 관심사·QA-13(top-line) altitude 명문 |
| Measurable | ○ | 신규/캐시 분리·tier·적중률 구체 |
| Realistic | ○ | 8k 현실적. 단 적중률 조건이 ★ 변별 폭 축소 |
| Consistent | ◎ | OI-9 통과 |

**verdict: Sound ◎ / KPI ○ · Low** (round-02 ○/○ Low 유지).

## Stage 2 권고
1. **[적중률 조건 silent cap · Low]** `조건: 적중률 ≥85% 고정`이 신규 토큰(cache-miss) 변별 폭을 스스로 좁힘을 ## 등급 척도에 명시 — 공정성과 변별력의 trade-off.
2. **[단일 출처 다양화 → Council · Low]** Requesty 코딩 에이전트 외 빌드 파이프라인 도메인 적중률·신규 토큰 분포 보강(round-05).
3. **[tier margin 명시 · Low]** 4k/6k/8k 2k 등간격의 margin 근거를 명시(현재 8k만 "복합 margin").
4. DP 역검토: DP-0001 2안 라우팅이 토큰 vs 비용 기준인지(OI-7), QA-13 top-line 이양 정합.
