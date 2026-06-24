# Counsel: QA-05 Efficiency — Agent 토큰·자원 효율 (round-02)

> refs-review: [round-02/review/QA-05-efficiency.md](../review/QA-05-efficiency.md) · 이양: [NQA-C](../review/NQA-C-cost-economy.md)
> 직전 counsel: [round-01/counsel/QA-05-efficiency.md](../../round-01/counsel/QA-05-efficiency.md) (채택 권장 — 캐시 분리집계, top-line→NQA-C)
> seats: 발의 **Seat 1**(Agentic Workflow) · 합의 **consensus**
> stance: **닫힘 확인(Med 해소) — 잔여 = NQA-C 동반 채택(C2)·비용 기준 라우팅 DP 위임**

## Reviewer 지적 요약
- round-02 verdict: **Sound ○ / KPI ○ · Low** (Med 해소). raw 8k 토큰캡 폐기 → `신규 토큰 ≤6k AND 캐시 토큰 별도 집계` + worker 가동률. 캐싱 역페널티 제거.
- 잔여(비-verdict): `$/완료모델` top-line **NQA-C 이양이 미채택 시 부유**(C2, QA-01 활용률과 동일 구조). DP-0001 라우팅 토큰 vs 비용 기준 미명시(OI-7). 캐시 적중률 보조 노출(Low).

## 개선안 (정의·KPI 기존→제안)
KPI 닫힘 — 새 KPI 없음. per-request altitude로 깨끗이 좁혀짐(top-line은 NQA-C 승격).
- **KPI**: 주 `신규 토큰 ≤6k AND 캐시 분리집계` 유지. **캐시 적중률을 보조 지표로 노출**(`≤6k` 합격이 적중률에 좌우 — 측정 정직성, Low). top-line → **NQA-C 동반 채택 확인**(미채택 시 부유, C2).
> ⚠️ `6k` 등은 "측정 가능 KPI의 모양" 예시값.

## 근거 (레퍼런스)
§4 **비용·토큰 효율(Cost/Efficiency)** 행.
- **raw token 아닌 $/task; cache read=정상가 10%(≈90%↓) 분리 집계** — 캐시로 비용 1/10인 전략이 벌점받지 않도록. https://platform.claude.com/docs/en/build-with-claude/prompt-caching · https://platform.claude.com/docs/en/about-claude/pricing
- worker 가동률·warm pool 재사용 = compute 효율의 runner 자리(SRE 활용도).

## PoC 증명법
### PoC-E1(R1 유지): prompt caching $/task·TTFT 절감 (비용 A/B)
- **가설**: "caching On/Off A/B로 신규/캐시 토큰 분리 집계 시 $/task·TTFT 절감이 측정 가능하며 적중률이 보조로 노출된다."
- **지표+합격선**: $/task 유의 감소 · TTFT 절감 · **캐시 적중률 보조 노출** · worker 가동률.
- **셋업**: prompt caching On/Off A/B + QA-04 OTel 토큰 계측(`gen_ai.usage.*`) 공유.
- **절차**: 동일 워크로드 caching On/Off → $/task·TTFT·적중률 비교.
- **합격(Exit)**: caching 절감 유의 ∧ 적중률 보조 노출 ∧ 신규 토큰 `≤6k` 측정 가능.
- **규모/기간**: QA-04 계측 후행 — 약 2일.
- **silent cap**: 캐시 적중률은 워크로드 유사성 의존(다양성 높은 실운영 절감폭 다름). compute 단가 = NQA-C 흡수 경계.

## DP·발표 영향
- **DP 위임 (OI-7)**: DP-0001 2안 라우팅을 **비용 기준 정렬** 명시(현재 토큰/비용 기준 미명시 — 비용 효율 KPI와 정합). NQA-C와 공유 항목. DP 디스커션 위임.
- **이양 동기화 (C2)**: `$/완료모델` top-line NQA-C 이양 — **NQA-C 채택 확인 필요**(QA-01 활용률과 묶어 처리).
- **번호/서사**: per-request 효율 = NQA-C(business top-line)와 altitude 분리. importance 변동 없음.
