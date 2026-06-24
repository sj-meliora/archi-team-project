# Counsel: QA-01 Scalability (round-02)

> refs-review: [round-02/review/QA-01-scalability.md](../review/QA-01-scalability.md) · 이양: [NQA-C](../review/NQA-C-cost-economy.md)
> 직전 counsel: [round-01/counsel/QA-01-scalability.md](../../round-01/counsel/QA-01-scalability.md) (채택 권장 — scaling efficiency + rate-limit 헤드룸)
> seats: 발의 **Seat 3**(Runner 인프라) · 합의 **consensus**
> stance: **닫힘 확인(High 해소) — 잔여 = NQA-C 동반 채택·DP 위임·예시값·Low cross-link**

## Reviewer 지적 요약

- round-02 verdict: **Sound ○ / KPI ○ · Low** (R01 High 해소). `N` placeholder 폐기 → `scaling efficiency ≥0.8`(USL), rate-limit 헤드룸·큐 p95 정착. 측정가능 KPI 닫힘.
- 잔여(전부 비-verdict): ① 활용률 NQA-C 이양이 **NQA-C 미채택 시 부유**(C2), ② rate-limit 헤드룸 측정단위(전역 vs 큐별)·USL 부하 단위(토큰 vs 모델 건수) Low 보강, ③ rate-limit headroom·admission control·bounded queue가 DP-0001/0004에 미명시(OI-7).

## 개선안 (정의·KPI 기존→제안)

KPI 닫힘 — 새 KPI 없음. Council 권고는 Low 보강·이양 동기화·DP 위임뿐.

- **정의**: throughput 차원·1차 병목 rate-limit 유지. **부하 단위를 "(정규화된) 동시 워크플로우 수 또는 토큰 처리량"으로 고정**(USL 동질 단위 전제, IR/Optimize/Quant/Compile 토큰 프로파일 상이 — Low).
- **KPI**:
  - 주 `scaling efficiency ≥0.8` 유지 — 부하 단위 명시 후 acceptance 재현 가능.
  - `rate-limit 헤드룸 ≥20%` — **헤드룸 보장 단위 명시**(전역 풀 vs 큐별 쿼터 배분, QA-06 외부 rate-limit 계정전역 silent cap과 cross-link, Low).
  - 활용률 → **NQA-C 동반 채택 확인**(미채택 시 부유, C2).

> ⚠️ `0.8·20%·5분`은 "측정 가능 KPI의 모양" 예시값.

## 근거 (레퍼런스)

§4 **확장성·처리량(Scalability)** 행.
- **Universal Scalability Law**(contention α·coherency β) — 동질 부하 단위 전제. https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/
- **backlog 기반 오토스케일(KEDA)** — 큐 깊이 신호. https://keda.sh/
- **rate-limit 헤드룸 = admission control + bounded queue**(SRE) — 폭주 시 p95 정의 보장. https://sre.google/workbook/implementing-slos/

## PoC 증명법

### PoC-S1·S2(R1 유지): scaling efficiency·rate-limit 헤드룸 (부하시험)
- **가설**: "부하 단위 고정 하에 scaling efficiency ≥◯, admission control로 throttle 0이 측정 가능."
- **지표+합격선**: 부하 2배→처리량 ≥1.8배(efficiency ≥0.8) · rate-limit 헤드룸 ≥◯%(429 throttle 0).
- **셋업**: k6/Locust + mock 4단계 파이프라인(부하 하네스 — S2·E2E1·R1 공유).
- **절차**: 부하 N배↑ → scaling efficiency · admission 거부율/backpressure 측정.
- **합격(Exit)**: efficiency 임계 통과 ∧ throttle 0 ∧ 부하 단위 명시로 재현 가능.
- **규모/기간**: 수십 VU, 약 2일.
- **silent cap**: mock 파이프라인 현실성(compute 분포 실측 의존). 외부 LLM 429는 계정전역(QA-06과 동일 한계). self-host vs 외부 API 가정에 따라 확장 전략 갈림.

## DP·발표 영향

- **DP 위임 (OI-7)**: DP-0001/0004에 **rate-limit headroom·admission control·bounded queue tactic** 미명시 — KPI 책임 설계 부재. bounded queue 없으면 `큐 p95 ≤5분`이 폭주 시 무한히 깨짐. DP 디스커션 위임.
- **이양 동기화 (C2)**: 활용률 NQA-C 이양 — **NQA-C 채택 확인 필요**(미채택 시 부유).
- **번호/서사**: throughput 축 = 발표 처리량 배수 앵커(목표 기간 합의 선결). importance 변동 없음.
