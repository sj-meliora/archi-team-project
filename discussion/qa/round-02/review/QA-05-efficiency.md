# Review: QA-05 Efficiency — Agent 토큰·자원 효율 (round-02 재평가)

> source: `context/qa/QA-05-efficiency.md` + `QAS-05-efficiency-token-budget.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound ○ / KPI △ — Med**
> verdict(round-02): **Sound ○ / KPI ○ — Low** — raw 8k 토큰캡 폐기·신규/캐시 분리집계로 캐싱 역페널티 제거(Med 해소). 잔존은 top-line 이양처(NQA-C) 미채택 의존·DP 토큰vs비용 역검토(OI-7) · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-05 = **[반영]** (총토큰≤8k → 신규/캐시 분리집계 + worker 가동률; top-line→NQA-C). 이월: DP-0001 토큰vs비용(OI-7); top-line NQA-C 전제.

## 원문 요약 (반영 후)
- **정의**: 모델 1건 총비용(토큰+compute) 최소화하되 prompt caching·context 압축을 페널티 없이 측정. 신규/캐시 토큰 분리집계가 핵심.
- **KPI**: 주 = `신규 토큰 ≤6k AND 캐시 토큰 별도 집계`. 보조 = 난이도별 tier(★) · `worker 가동률/compute 비용`. 이양 = `완료 모델당 비용($)` → NQA-C 승격.
- **QAS-05**: Response·Measure를 신규/캐시 분리·compute 효율로 동기화.

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01의 핵심 지적("raw 토큰 캡이 prompt caching을 역페널티")이 정확히 반영됐다. `신규 토큰 ≤6k AND 캐시 토큰 별도 집계`는 캐시로 비용 1/10인 좋은 전략이 "토큰 많이 썼다"고 벌점받던 구조를 제거했다. prompt caching(cache read ≈ 정상가 10%)을 분리 집계하는 건 현대 agentic 효율의 정석. **닫혔다.**

- **잔여(silent cap)**: 캐시 적중률이 워크로드 유사성에 의존 → **다양성 높은 실운영에선 절감폭이 다를 수 있음**이 검증 전략에 명시됨. 즉 `신규 토큰 ≤6k`의 합격 여부가 캐시 적중률에 좌우 → KPI를 "캐시 적중률 가정 하에"로 조건화하거나, 적중률 자체를 보조 지표로 노출 권고(Low). 신규 결함 아님 — 측정 정직성 보강.
- **compute 효율의 측정 단가 부재**: `worker 가동률 / task당 compute 비용`은 **실제 인프라 단가가 있어야 비용으로 환산**되는데 단가가 silent cap. 가동률(%)은 단가 없이 측정 가능하나 "compute 비용($)"은 추정 — 이건 NQA-C로 흡수되는 경계와 연결.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **measurable 회복**: `총 토큰 ≤8k`(입력+출력 미명시 + 빌드로그 비현실 + 캐싱 역페널티)가 `신규 토큰 ≤6k + 캐시 분리(입력+출력 명시)`로 교정. 집계 단위가 명확해져 측정 가능. **합격.**
- **altitude 정리 — round-02 핵심 구조 쟁점**: round-01 지적("top-line은 `$/완료모델`인데 KPI는 요청당 토큰뿐")에 대해 **top-line을 NQA-C로 승격**해 응답했다. 이로써 QA-05는 per-request altitude로 깨끗이 좁혀졌다(Sound ○ 강화). **그러나 이 이양은 NQA-C 신설을 전제**하고 NQA-C는 [이월](미채택, OI-8)이다. 즉:
  - QA-05 본문은 "`$/완료모델`은 **NQA-C로 이양**(본 QA에 남기지 않음)"이라 적었다.
  - NQA-C가 정식 채택 안 되면 **top-line 비용 KPI가 어디에도 살아 있지 않다**(QA-05엔 명시적으로 뺐고 NQA-C는 임시 ID).
  - → **교차 의존(C2): QA-01의 활용률 이양과 동일 구조** — NQA-C 동반 채택이 닫힘 전제. 미채택이면 QA-05 본문에 "NQA-C 채택 전까지 잠정 보유"로 둔 ROI 수치가 부유(浮遊).
- consistency ○: QAS-05 Measure 동기화 확인.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **compute 효율 통합**(round-01 렌즈3: "활용률은 runner 효율 지표가 올바른 자리") → `worker 가동률 / task당 compute 비용`으로 반영. warm pool 재사용·cold-start 절감이 보조 KPI로. **방향 양호.**
- **잔여(DP 위임, OI-7)**: 주 KPI를 **DP-0005 2안 공유 캐시 + DP-0001 2안 동적 풀**에 귀속시키나, OI-7이 명시하듯 **DP-0001(작업별 최적 agent)이 토큰 기준인지 비용 기준인지 미명시**다. 비용 효율 KPI인데 라우팅이 토큰 기준이면 정렬이 어긋남 → "비용 기준 정렬" DP 디스커션 위임.
- **runner 측 KPI**(재확인): worker 가동률 vs headroom(QA-01 헤드룸과 상충 관리), cache hit율(보조 노출 권고), warm pool 적중률·cold-start 비율, 배칭 효율.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | ○ | **○** | per-request altitude로 좁혀짐(top-line은 NQA-C로 승격). 캐싱 효율 단일 관심사 |
| KPI가 측정 가능한가 | △ | **○** | raw 8k 폐기 → 신규/캐시 분리집계, 집계 단위 명확. acceptance 가능 |
| KPI가 현실적/적절한가 | △ | **○** | 캐싱 역페널티 제거. 단 캐시 적중률 의존·compute 단가는 Low 보강 |
| 정의↔KPI↔QAS 일치 | ○ | **○** | QAS-05 Measure 동기화. 단 top-line 이양은 NQA-C 채택 의존(C2) |

**verdict 변화: Sound ○→○ / KPI △→○ · severity Med→Low.** round-01 Med의 근거(8k 비현실·캐싱 역페널티·altitude)는 해소됨. NQA-C 동반 채택은 닫힘 전제(C2, Low).

## Stage 2 권고 (round-02)

- **(NQA-C 동반 채택, OI-8 — C2)** `$/완료모델` top-line 이양처(NQA-C) 정식 채택 확인. 미채택이면 QA-05 본문 "잠정 보유" ROI 수치가 부유 → 채택 결정과 동기화. QA-01 활용률 이양과 묶어 처리.
- **(DP 디스커션 위임, OI-7)** DP-0001 2안 라우팅을 **비용 기준 정렬**로 명시(현재 토큰/비용 기준 미명시). 비용 효율 KPI와 라우팅 기준 정합.
- **(캐시 적중률 노출, Low)** `신규 토큰 ≤6k` 옆에 **캐시 적중률을 보조 지표로 노출** — 합격이 적중률에 좌우됨을 정직하게 드러냄(다양성 높은 실운영 대비).
- **(예시값 확정)** `6k·tier 4~8k`·compute 단가는 prompt caching A/B로 확정.
