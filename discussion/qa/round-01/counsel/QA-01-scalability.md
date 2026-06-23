# Counsel: QA-01 Scalability

> refs-review: round-01/review/QA-01-scalability.md · report.md(C1·C3·렌즈3 횡단)
> seats: 발의 Seat 3(Runner 인프라) · 합의 consensus (Seat 1·2 보강)
> stance: 채택 권장 (KPI 전면 재설계)

## Reviewer 지적 요약
- KPI ① `시간당 완료 모델 수 ≥ N` 의 **N이 미정의 placeholder** → acceptance test 불가 (렌즈2, C1).
- KPI ② `자원 활용률 ≥ 70%` 는 **확장성 지표가 아니라 비용/효율 지표**이며, 항상 70%를 유지하면 burst headroom이 사라져 **가용성과 상충** (렌즈2).
- 진짜 병목은 compute가 아니라 **LLM rate-limit(TPM/RPM)·동시 세션** (렌즈1). 오토스케일 신호도 CPU가 아니라 **backlog(큐 깊이)**여야 함 (렌즈3).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "모델·워크플로우 수 증가에 비례해 확장."
- 제안: "피크 부하에서 자원을 비례 투입해 **처리량을 선형에 가깝게 유지**하며, 1차 병목은 **LLM rate-limit(TPM/RPM) 용량**으로 본다. 자원 가산은 **backlog(큐 깊이·대기시간)** 신호로 구동한다." — *throughput 차원으로 altitude 고정* → QA-07/08과 경계 분리.

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | 시간당 완료 모델 수 ≥ N | **scaling efficiency ≥ 0.8** (부하 2배 투입 시 처리량 ≥ 1.8배) | USL 기반. `N` 절대값 placeholder 제거 |
| ②(보조) | — | **베이스라인 대비 처리량 ≥ ◯배** (수작업 대비) | 발표 서사용 절대 앵커. ◯는 overview "모델 140개+" × 목표기간으로 산출(PoC-S1) |
| ③ | 자원 활용률 ≥ 70% | **(삭제·이전)** → QA-05/NQA-C(cost) 계열로 이동 | 활용률은 cost 지표. report.md 렌즈3·C5 일치 |
| ④ | — | **rate-limit 헤드룸 ≥ ◯%** (피크 시 TPM/RPM 여유) | agentic 고유 병목(C4). ◯는 PoC-S2로 확정 |
| ⑤ | — | **큐 대기 p95 ≤ ◯분** (backlog 신호의 SLI) | 오토스케일 트리거 SLI |

> ◯는 모두 placeholder가 아니라 **PoC 산출값으로 확정**되는 자리 — 산출 절차를 아래 PoC에 명시.

## 근거 (레퍼런스)
- **scaling efficiency / USL**: Universal Scalability Law는 contention(α)·coherency(β)로 scale 한계를 모델링한다. "부하 N배 → 처리량 N배 유지"의 효율을 정량화하는 정통 척도. (§4 확장성 유형) — https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/
- **backlog 기반 오토스케일**: CPU가 아닌 **큐 깊이**로 scale-out하는 것이 이벤트/LLM-bound 워크로드의 필드 표준(KEDA). (§4) — https://keda.sh/
- **percentile SLI(큐 대기 p95)**: 평균이 아닌 p95/p99로 지연 SLI를 잡는 SRE 표준. (§4 지연) — https://sre.google/workbook/implementing-slos/ , https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view
- **rate-limit 헤드룸**: 외부 LLM 제공자의 TPM/RPM 한도가 agentic 시스템의 실질 처리량 상한이라는 것은 Anthropic 등 제공자 rate-limit 문서·운영 관행에서 표준. (라이브러리 외 보강) — https://platform.claude.com/docs/en/api/rate-limits
- ⚠️ 레퍼런스의 효율 0.8·헤드룸% 등 수치는 **패턴 정당화용**이며, 우리 시스템 합격선은 PoC로 직접 확정한다.

## PoC 증명법

### PoC-S1: scaling efficiency가 측정·달성 가능함을 보인다
- **가설**: "부하 2배 투입 시 처리량 ≥ 1.8배(efficiency ≥ 0.8)가 측정 가능하고, 우리 runner가 이를 만족한다."
- **지표**: efficiency = (처리량_2x / 처리량_1x) / 2. 합격선 ≥ 0.8. + 베이스라인 처리량(모델/hr) → 절대 앵커 ◯ 산출.
- **셋업**: 모델 컴파일 파이프라인 mock worker(LLM 호출은 stub/지연 모사) + k6/Locust로 동시 워크플로우 수를 N→2N 계단 부하. KEDA 큐-깊이 스케일러 연결.
- **절차**: ① 동시 WF=N 정상부하 처리량·p95 측정 → ② 2N으로 올리고 worker/큐 스케일 → ③ efficiency·큐 대기 p95 산출.
- **합격(Exit)**: efficiency ≥ 0.8 이 재현되고, 큐 대기 p95가 스케일 후 임계 내로 복귀.
- **규모/기간**: 동시 WF 수십, 부하 2계단, 약 1~2일.
- **리스크/한계(silent cap)**: LLM 호출을 stub으로 모사하므로 **실제 제공자 rate-limit 거동·토큰 변동성은 미관측** → PoC-S2가 보완. coherency(β) 비선형 구간은 부하 2배만으론 안 보임(고배율 미검증).

### PoC-S2: rate-limit 헤드룸이 진짜 병목임을 노출한다
- **가설**: "compute가 아닌 LLM TPM/RPM이 처리량 상한이며, 헤드룸 ◯%를 SLI로 잡을 수 있다."
- **지표**: 피크 시 TPM/RPM 사용량 대비 한도 여유(%) + 429/throttle 발생률. 합격선: 헤드룸 ≥ ◯%에서 throttle 0.
- **셋업**: 실제(또는 한도 모사) LLM 엔드포인트에 동시 agent 세션 점증, token-bucket admission control On/Off.
- **절차**: ① 세션 점증으로 429 발생점 탐지 → ② backpressure(bounded queue) 켜고 무한재시도 차단 → ③ 헤드룸-처리량 곡선 작성.
- **합격(Exit)**: admission control On에서 throttle 폭주 없이 처리량 유지, 헤드룸 ◯% 임계 확정.
- **규모/기간**: 단일 엔드포인트, 약 1일.
- **리스크/한계**: self-host GPU 풀 시나리오는 미검증(rate-limit이 GPU 큐로 치환됨 — 별도 PoC 필요).

## DP·발표 영향
- **DP 연결**: DP-0001(2안 Dynamic Agent Pool)·DP-0004(A5/A8 일회용 scale-out)가 backlog 신호·worker class 분리를 실현. **두 DP 모두 rate-limit 차원을 명시적으로 다루지 않음** → DP-0001에 "admission control / TPM 헤드룸" tactic 보강 권고(역검토 항목).
- **번호/서사**: KPI ③(활용률) 이전은 **NQA-C(Cost) 채택과 연동** — NQA-C 채택 전엔 활용률을 QA-05로 임시 이전. 발표에서 QA-01은 "처리량 선형성 + LLM 병목 직시"로 재포지셔닝(일반 분산시스템 어휘 탈피).
- report.md C3(Performance 통합): QA-01=throughput 축으로 고정해 QA-07(per-node)·QA-08(E2E)와 3분할 정렬.
