# Counsel: QA-06 Reliability — Workflow 간 독립성 보장

> refs-review: round-01/review/QA-06-reliability-workflow.md · report.md(렌즈3 횡단·C3·C4)
> seats: 발의 Seat 3(Runner 인프라) · 합의 consensus (Seat 1 토큰 쿼터 격리, Seat 2 경계 명문화)
> stance: 채택 권장 (이미 건강 — 자원-쿼터 격리 KPI 추가 + QA-02 경계 명문화)

## Reviewer 지적 요약
- **이 세트에서 드물게 KPI가 건강** — `타 WF 중단 ≤1%`, `latency ≤10%` 구체·측정 가능 (렌즈2).
- **QA-02와 경계 중복(C3)**: 둘 다 fault — 명문화 필요(QA-02=복구, QA-06=격리/blast radius).
- **자원-쿼터 격리 누락**: agentic 진짜 공유 장애 도메인은 노드가 아니라 **LLM rate-limit 풀·공유 캐시** — 한 WF 폭주가 전체 토큰 예산을 빨아들임 (렌즈1).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "특정 Workflow 장애가 타 Workflow에 무영향."
- 제안: "특정 Workflow의 장애·**자원 폭주**가 타 Workflow의 실행·지연·**토큰/rate-limit 쿼터**에 무영향(blast-radius 봉쇄)." (QA-02=복구와 구분되는 격리 QA임을 명시)

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | 타 WF 중단 ≤ 1% | **유지** | 건강 |
| ② | latency 증가 ≤ 10% | **유지** | 건강 |
| ③ | — | **자원 격리: 단일 WF의 토큰/rate-limit 소비가 타 WF 쿼터 침범 = 0** (또는 WF별 쿼터 보장률) | 신규. PoC-R1 |
| ④ | — | **공유 캐시 오염 전파 = 0** (한 WF 오염 데이터가 타 WF로 미전파) | DP-0005 격리 경계 |

## 근거 (레퍼런스)
- **bulkhead·큐별 동시성 제한·token-bucket·fair queueing**: WF 타입/테넌트별 전용 큐·worker pool + 큐별 max concurrency + 공유 LLM 풀의 WF별 token-bucket rate limiter로 noisy-neighbor 차단. (§4 격리·멀티테넌시) — https://keda.sh/ , https://sre.google/workbook/implementing-slos/
- **circuit breaker**: 장애 WF는 차단기로 격리해 재시도 폭주 차단(렌즈3). (SRE 패턴)
- ⚠️ 1%·10%는 기존 값 유지(건강), ③④ 임계는 PoC로 확정.

## PoC 증명법

### PoC-R1: noisy-neighbor 주입으로 쿼터 격리를 증명한다 (격리 주입)
- **가설**: "한 WF를 폭주(무한 재시도·토큰 과소비)시켜도 타 WF의 중단 ≤1%·latency ≤10%·쿼터 침범 0이 유지된다."
- **지표**: 타 WF 중단율, latency 증가율(p95), 타 WF의 토큰/rate-limit 쿼터 침범 건수(=0), fairness 지수.
- **셋업**: 다수 WF 동시 실행 + WF별 전용 큐·token-bucket rate limiter + 한 WF에 폭주 주입(무한루프·대량 토큰).
- **절차**: ① 정상 다중 WF baseline(중단율·latency·쿼터) → ② 한 WF 폭주 주입 → ③ 타 WF 지표 변화·쿼터 침범 측정. rate limiter On/Off 비교.
- **합격(Exit)**: 폭주 중에도 타 WF 중단 ≤1%·latency ≤10%·쿼터 침범 0.
- **규모/기간**: WF 수십 + 폭주 1건, 약 1~2일.
- **리스크/한계(silent cap)**: 외부 LLM 제공자 측 rate-limit이 계정 전역이면 token-bucket이 client-side 분배만 보장(제공자측 공유 한도 자체는 못 늘림) → 가정 명시 log. 공유 캐시 오염(④) 시나리오는 별도 데이터 주입 필요.

## DP·발표 영향
- **DP 연결**: DP-0004(A5/A8 invocation/단계 bulkhead)·DP-0005(1안 로컬 캐시=오염 격리, 2안 공유 캐시=전파 위험)가 ①②④에 직결. **DP-0005 2안 채택 시 ④(오염 전파 0)가 R-1 위험과 직접 충돌** → 무효화·읽기전용 계층화 권고와 일관.
- **경계(C3)**: QA-02(복구)↔QA-06(격리) cross-link을 양쪽 본문에 박아 grep 추적. QA-02 counsel과 짝.
- **rate-limit 격리는 QA-01(헤드룸)과 공유 인프라** — token-bucket·admission control이 두 QA를 동시 실현.
