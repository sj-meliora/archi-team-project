# Counsel: QA-03 Controllability

> refs-review: round-04/review/QA-03-controllability.md
> seats: 발의 Seat 3 (cancel·폴링 밀도) · 합의 consensus
> stance: **채택 권장 — 세트 모범(규칙2·4·5 동시), ★★★ 15초 근거 다양화 + 롤백 외부 정합성 한 줄만 보강**

## Reviewer 지적 요약
graceful stop ≤30초 흡수 재배치는 규칙2·4·5를 모두 정확 적용한 **세트 모범**(KPI △→○ 개선). 잔여(Low): (1) **★★★ 15초가 k8s 30초 단일 출처에서 ★★★·★★☆ 둘 다 파생**(권고 1), (2) 롤백(외부 시스템 상태 되돌림) 시간이 인프라 grace와 무관하게 더 걸릴 수 있음(권고 2·이미 silent cap).

## 개선안 (정의·KPI 기존→제안)
### (A) ★★★ 15초 근거 다양화 (권고 1)
- ★ 노트 기존: k8s grace 30초 단일 출처에서 15초(이등분)·30초(상한) 파생. → 제안: **`15초 = k8s grace 이등분 + 폴링 밀도 촘촘 가정`임을 명시**하고, non-k8s control-plane(Temporal heartbeat cancel 전파 15~20초)을 보강 출처로 병기. 롤백 비용 큰 케이스는 별도 변수로 분리.
### (B) 롤백 외부 정합성 표 한 줄 (권고 2)
- ★ 급간 표에 `★ 급간은 인프라 grace(cancel 전파)는 반영하나 외부 시스템 롤백(부분 롤백 정합성)은 별도 변수 — silent cap` 한 줄 추가(이미 silent cap에 있으나 표에도).

## 근거 (레퍼런스 + 검증 결과)
**C4 — 인프라 grace 정합(apples 우려 최저)**: k8s graceful shutdown(terminationGracePeriod 기본 30초)·Temporal heartbeat cancel 전파는 인프라 표준에 직접 정합 — 본 케이스는 C1 apples-to-apples 우려가 세트에서 가장 낮음(인용 출처 = 우리 control-plane과 동일 메커니즘). durable execution cancel은 [Temporal](https://docs.temporal.io/temporal)(Council.md §4 가용성·격리 공유). 1차 대조 미완이나 인프라 표준이라 신뢰도 높음.

## PoC 증명법
### PoC-03: graceful stop+롤백 시간 (장애주입/취소 주입 아키타입)
- 가설: "graceful stop+롤백 ★ 경계(15/30초)가 폴링 밀도별로 단조 감소하고 hard-kill 폴백이 유한 상한 보장한다."
- 지표: cancel 발행→graceful stop+롤백 완료 분포, ack 시간, hard-kill 폴백 상한. 합격선: ack ≤5초·HITL 100·runaway 100 게이트 하 ★ 경계 변별.
- 셋업: cancel token 폴링 밀도 가변 모델 + heartbeat 미설정 activity(전파 0%) 케이스.
- 절차: ① 폴링 밀도 sweep → graceful stop 분포. ② heartbeat 미설정 시 hard-kill 폴백 발동·유한 상한 확인.
- 합격 기준: 폴링 촘촘할수록 ≤15초, hard-kill이 항상 유한 상한.
- 규모/기간: 폴링 밀도 3단계, ~0.3일.
- 리스크/한계(silent cap): 외부 시스템 롤백은 인프라 grace와 독립으로 더 걸림.

## DP·발표 영향
- DP-0002/0003의 runaway cap·graceful stop tactic 미명시 + eval/검증 서브시스템 DP(②-2 하네스, OI-7). ②-2 ↔ QA-06 하네스(OI-8) 교차 의존 정상 트래킹.
