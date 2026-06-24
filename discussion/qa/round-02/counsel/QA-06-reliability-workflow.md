# Counsel: QA-06 Reliability — Workflow 간 독립성 보장 (round-02)

> refs-review: [round-02/review/QA-06-reliability-workflow.md](../review/QA-06-reliability-workflow.md)
> 직전 counsel: [round-01/counsel/QA-06-reliability-workflow.md](../../round-01/counsel/QA-06-reliability-workflow.md) (채택 권장 — 쿼터 격리 KPI 추가)
> seats: 발의 **Seat 3**(Runner 인프라) · 합의 **consensus**
> stance: **닫힘 확인(Med 해소·세트 모범) — 잔여 = DP-0004/0005 격리 역검토 위임**

## Reviewer 지적 요약
- round-02 verdict: **Sound ○ / KPI ○ · Low** (Med 해소). 건강 KPI 유지 + 자원-쿼터 격리(`쿼터 침범0` token-bucket)·캐시오염(`전파0`) 추가 + QA-02 경계 양방향 명문화로 **C3 닫힘. 세트 모범.**
- 잔여(비-verdict): DP-0004/0005 격리 역검토(OI-7), 외부 rate-limit 계정전역 silent cap(QA-01 헤드룸 단위와 연결).

## 개선안 (정의·KPI 기존→제안)
KPI 닫힘 — 새 KPI 없음. blast-radius 봉쇄·진짜 공유 장애 도메인(LLM rate-limit 풀·공유 캐시) 정의 유지. QA-01 rate-limit 헤드룸 단위(전역 vs 큐별)와 **cross-link 보강**(외부 rate-limit 계정전역 silent cap 공유).
> ⚠️ `1%·10%·0`은 "측정 가능 KPI의 모양" 예시값.

## 근거 (레퍼런스)
§4 **격리·멀티테넌시(Isolation)** 행.
- **bulkhead·큐별 동시성 제한·token-bucket rate limiter·fair queueing** — 한 WF 폭주가 타 WF 쿼터 침범 못하게. https://keda.sh/ · https://sre.google/workbook/implementing-slos/

## PoC 증명법
### PoC-R1(R1 유지): noisy-neighbor 쿼터 격리 (격리 주입)
- **가설**: "한 WF 폭주 주입 시 타 WF 중단·latency 증가가 임계 이내, token-bucket로 쿼터 침범 0이 측정 가능."
- **지표+합격선**: 타 WF 중단 ≤◯% · latency 증가 ≤◯% · 쿼터 침범 0 · 캐시 오염 전파 0.
- **셋업**: 부하 하네스(k6/Locust — S1·E2E1 공유) + 한 WF noisy-neighbor 주입 + token-bucket 쿼터.
- **절차**: 폭주 WF 주입 → 타 WF 건강(중단·latency)·쿼터 침범·캐시 오염 측정.
- **합격(Exit)**: 타 WF 건강 임계 이내 ∧ 쿼터 침범 0 ∧ 오염 전파 0.
- **규모/기간**: 약 2일.
- **silent cap**: 외부 LLM rate-limit는 계정전역이라 격리 한계(공유 풀 침범은 token-bucket로 내부만 보호). 공유 캐시 무효화·읽기전용은 DP-0005 2안 충돌 가능.

## DP·발표 영향
- **DP 위임 (OI-7)**: DP-0004/0005 격리 역검토 — 공유 캐시 오염 격리(무효화·읽기전용)가 DP-0005 2안 채택 시 R-1과 충돌. rate-limit headroom·admission control은 QA-01과 공유 항목. DP 디스커션 위임.
- **번호/서사**: 세트 모범(C3 닫힘 완료) — 발표에서 격리 설계의 acceptance 시연 사례. importance 변동 없음.
