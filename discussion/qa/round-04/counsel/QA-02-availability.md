# Counsel: QA-02 Availability

> refs-review: round-04/review/QA-02-availability.md
> seats: 발의 Seat 3 (durable execution·lease) · 합의 consensus
> stance: **조건부 채택 — ★★★ main 축 정의 고정 + 외부 outage silent cap + lease trade-off 명시**

## Reviewer 지적 요약
failover 1분→4분 재배치는 게이트/별점 분리 모범. red-team(Low): (1) **★★★ 10초가 "재스케줄 ~0.5초"와 "컨테이너 재기동 ~10초"를 OR로 섞은 단위 혼동**(권고 1), (2) 외부 LLM outage 길이(수십분~시간)가 ★ 급간에 미반영(권고 2), (3) durable execution 0.5~20초·DORA MTTR <1hr 단위 차이(C1), (4) lease 파라미터가 ★를 좌우.

## 개선안 (정의·KPI 기존→제안)
### (A) main 축 정의 고정 (권고 1)
- ★ 노트 기존: "재스케줄 0.5초 vs 컨테이너 재기동 10초"를 OR로 10초. → 제안: **"재기동" = `lease 만료 후 다른 worker가 체크포인트부터 재개 완료까지`로 정의 고정**(0.5초 재스케줄과 10초 cold 재기동 분리, ★★★ 경계 의미 명확화).
### (B) 외부 outage silent cap (권고 2)
- `★ 급간(내 노드 재기동 시간)은 외부 LLM outage 길이(분~시간)를 안 잰다 — 외부 장애는 자동 재개율(≥95%) 보조 KPI로만 다룸. ★★★(≤10초)를 받아도 외부 LLM 30분 outage면 가용성 무너짐.`
### (C) lease trade-off (권고 3)
- `빠른 lease(10초)가 손실 게이트(손실=0)와 trade-off일 수 있음 — ★ 급간이 lease 값 선택을 보상하지 않도록 손실률 독립 확인`.

## 근거 (레퍼런스 + 검증 결과)
**C4 — 단위 차이**: durable execution 재개(0.5~20초)는 [Temporal](https://temporal.io/blog/what-is-durable-execution)(event history·exactly-once), DORA elite MTTR(<1hr)은 인시던트 단위 — **서로 다른 단위**임을 노트 명시(표가 나란히 놓아 혼동 여지). 둘 다 본 검증에서 1차 대조는 미완 → "출처 유효, 단위 구분 명시"로 처리(Council.md §4 가용성).

## PoC 증명법
### PoC-02: 재기동 시간 + 무손실 (장애주입 chaos 아키타입)
- 가설: "재기동 시간 ★ 경계(10초/1분/4분)가 chaos 주입으로 측정되고 손실=0 게이트와 독립이다."
- 지표: lease 만료→체크포인트 재개 완료 시간 분포, in-flight 손실 건수, 멱등성. 합격선: 손실=0·멱등=100% 하 ★ 경계 변별.
- 셋업: durable execution 엔진(event sourcing), 노드 강제종료 chaos.
- 절차: ① 노드 강제종료. ② 재개 시간·손실 측정. ③ lease 값 sweep으로 trade-off 노출.
- 합격 기준: 손실=0 유지하며 재기동 시간 분포가 lease별 단조.
- 규모/기간: chaos 수십 회, ~0.5일.
- 리스크/한계(silent cap): 외부 LLM outage는 자동 재개율로만, ★ 급간 밖.

## DP·발표 영향
- DP-0002/0003의 외부 LLM degradation(backoff·폴백) 미명시(OI-7).
