# Review: QA-03 Controllability

> source: context/qa/QA-03-controllability.md + QAS-03-controllability-stop.md
> verdict: **Sound ◎ / KPI ○** — graceful stop ≤30초 흡수 재배치는 규칙2·4·5를 모두 정확히 적용한 세트 모범. 잔여는 ★★★ 15초 경계가 단일 출처(k8s 30초)에서 파생된 점 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(graceful stop ≤15/30초 재배치) 검증 무게중심.

## 원문 요약
- 정의: 최소권한 allowlist + 고위험 HITL + runaway cap 즉응. Controllability ⊂ Security(QA-06).
- 헤드라인: `중단 ack ≤5초 AND graceful stop+롤백 ≤30초`(2-index).
- ★ 급간: main = graceful stop+롤백 시간(역방향). ★★★ ≤15초 / ★★☆ 15~30초 / ★☆☆ >30초+hard-kill 폴백. 조건: ack ≤5초. 게이트: HITL 100·runaway 100·②-1 차단(0건/100% 절대형).

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
runaway cap·협조적 취소·hard-kill 폴백을 게이트로 둔 것은 agentic 고유 리스크(폭주)를 정확히 반영한 모범. ★ 급간 근거가 k8s grace(30초)·Temporal heartbeat cancel 전파(15~20초)로 **인프라 표준에 직접 정합**해 apples-to-apples 우려가 가장 낮은 케이스다. 다만 "heartbeat 미설정 activity는 cancel을 못 받음(전파 0%)"을 ★☆☆ 합격 조건(hard-kill 폴백 필수)으로 흡수한 것은 정확 — agentic 통찰. 보강 요청: ★★★ 15초가 "LB 드레인 5초 + cleanup 15~25초"에서 나왔는데, **롤백(외부 시스템 상태 되돌림)**은 인프라 grace와 무관하게 더 오래 걸릴 수 있다(부분 롤백 정합성, silent cap에 이미 명시).

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙4·5 동시 적용 모범**: ack(변별력 낮은 1차 응답)를 조건으로 빼고(규칙4), HITL/runaway/②-1을 게이트로 빼고(규칙5), gradable한 graceful stop+롤백 시간에만 별점. **세트에서 가장 정교한 분해.** ◎.
- **급간 reasonableness**: ★★★ ≤15초 / ★★☆ 15~30초 / ★☆☆ >30초. **등간격(15초 폭) + ★☆☆ 오픈** — k8s 30초를 중심으로 절반(15초)을 ★★★ 경계로. 자의적 등간격 아니라 30초 표준을 이등분한 합리적 배치. ○.
- **하한 보정 정합(OI-9)**: §측정(38행 보정노트), 등급표, 변경이력 3블록, counsel(30s 흡수), QAS-03(`≤15초/≤30초/>30초+hard-kill`) **5곳 일치**. glossary 없음. **OI-9 통과.**
- **변별력**: 구 ≤30초를 ★☆☆ 게이트로 두면 ★★☆ 사문화였음(보정 사유 정확). 신 재배치(30초를 ★★☆ 상한으로)로 ★★☆ 부활. ★☆☆을 "30초 초과 + hard-kill 폴백"으로 둔 것은 **합격 진입선을 "유한시간 정지 보장"이라는 질적 조건으로 전환**한 영리한 처리 — 사문화 없음. ◎.
- **단, ★★★ 15초 단일 출처**: k8s 30초 하나에서 ★★★(절반)·★★☆(상한) 둘 다 파생. 다른 control-plane(non-k8s)·롤백 비용이 큰 경우엔 15초가 자의적. → Low.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
graceful stop = cancel token 폴링 밀도의 함수. ★★★(≤15초)는 "폴링 지점을 촘촘히 둔 잘 설계된 control-plane"의 도달선 → **★ 급간이 폴링 밀도 설계를 직접 보상**(설계 대안 변별로 적합). hard-kill 폴백을 ★☆☆ 필수로 둬 "무한 정지" 불합격을 막은 것은 durable execution 관점 정확. runner KPI: `cancel 발행→graceful stop 완료 분포가 폴링 밀도별로 단조 감소` + `hard-kill 폴백이 항상 유한 상한 보장`.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | 제어성 단일 관심사·Controllability ⊂ Security 명문·게이트/별점/조건 3분해 모범 |
| Measurable | ○ | ack·graceful stop·게이트 모두 구체. 롤백 시간 외부 정합성은 silent cap |
| Realistic | ○ | 15/30초가 k8s·Temporal 정합, ★★☆ 부활로 변별력 회복 |
| Consistent | ◎ | 5곳 일치(OI-9 통과), ②-1/②-2 분리 정합 |

**verdict: Sound ◎ / KPI ○ · Low** (round-02 ◎/△ Med → ★ 신설로 **KPI △→○ 개선**, contention ②-1 선닫힘 + ★ 재배치로 Med→Low).

## Stage 2 권고
1. **[★★★ 15초 근거 다양화 · Low]** k8s 30초 단일 출처 외에 non-k8s control-plane·롤백 비용 큰 케이스 보강(round-05 Council) — `제안: 15초는 "k8s grace 이등분 + 폴링 밀도 촘촘" 가정임을 명시, 롤백 외부 정합성은 별도 변수`.
2. **[롤백 외부 정합성 · Low]** ★ 급간이 인프라 grace는 잘 반영하나 외부 시스템 롤백 시간은 미반영 — 이미 silent cap에 있으나 ## 등급 척도 표에 한 줄 더.
3. DP 역검토: DP-0002/0003의 runaway cap·graceful stop tactic 미명시 + eval/검증 서브시스템 DP(②-2 하네스, OI-7).
4. ②-2 ↔ QA-06 하네스(OI-8) 교차 의존은 이번 라운드 범위 밖(정상 트래킹).
