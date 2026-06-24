# Applier 반영 보고서 — round-01 (concept=qa)

> 반영 대상: round-01의 [review](../review/report.md) + [counsel](../counsel/counsel.md) + [_feasibility-filter](../counsel/_feasibility-filter.md)
> 반영 일자: 2026-06-24 · 반영 범위: QA-01~10 + NQA-A/B/C(신설) · 원본: `context/qa/`
> 다음 라운드(round-02) Reviewer는 이 보고서를 입력으로 읽고, 각 지적의 처리를 확인해 verdict 변화를 재평가한다(Reviewer §8 절차 0).

## 1. 한눈에 (항목별 처리 요약)

| ID | review verdict | 처리 등급 | 무엇을 바꿨나 (1줄) | 이월/재검증 |
|---|---|---|---|---|
| QA-01 Scalability | △ / ✕ High | [반영] | 완료모델≥N·활용률 폐기 → scaling efficiency≥0.8 + rate-limit 헤드룸 + 큐 p95 | DP-0001/0004 rate-limit 역검토(OI-7); 활용률→NQA-C |
| QA-02 Availability | ○ / △ High | [반영] | MTTR 단독 → 재기동≤1분·손실0 + 가용률 99.5% + 외부LLM 재개 + 멱등100% | DP-0002/0003 외부 LLM degradation(OI-7) |
| QA-03 Controllability | ◎ / △ Med | [반영] + 일부 [발표 서사] | 수용≤5초 → ack/안전정지 분리 + 위반0·runaway cap·HITL | 적대적 eval 실측 [발표 서사]; DP runaway cap(OI-7) |
| QA-04 Observability | ○ / △ Med | [반영] | 재구성95% → trace 완전성(span 정의)·event-history·MTTD·결정당 비용 | DP-0003 span 보장 역검토(OI-7) |
| QA-05 Efficiency | ○ / △ Med | [반영] | 총토큰≤8k → 신규/캐시 분리집계 + worker 가동률; top-line→NQA-C | DP-0001 토큰vs비용(OI-7); top-line NQA-C 전제 |
| QA-06 Reliability-WF | ○ / ○ Med | [반영] | 건강 KPI 유지 + 쿼터 침범0·캐시오염0 + QA-02 경계 명문화 | DP-0004/0005 격리 역검토(OI-7); 외부 rate-limit 계정전역 silent cap |
| QA-07 Performance-Agent | △ / ✕ High | [반영] | "최대화" 폐기 → 노드타입별 speedup(first-pass 게이트)·커버리지·오버헤드 | NQA-B golden 게이트 전제; DP-0001 품질게이트 무연결(OI-7) |
| QA-08 Performance-E2E | △ / ✕ High | [반영] | 정의↔KPI 불일치 복구 → E2E latency p50/p95·throughput; 전달5% 강등 | DP-0004 5% 산식 실측; importance 상향 여지 |
| QA-09 Reliability-일관성 | △ / △ Med | [반영] + 일부 [발표 서사] | 결정성 폐기 → 캐시우회 pass^k + 유효-결정률 재프레이밍 | pass^k 실측 [발표 서사]; NQA-B 유효결정률 전제; importance 상향 여지 |
| QA-10 Maintainability | ○ / ○ Low | [반영] | 평균CIS → CIS p95 + prompt/모델 교체·온보딩 축 | DP-0004/0005 교체내성 역검토(OI-7) |
| **NQA-A Security/Safety** | 신설·강력권장 | [반영](신설) + eval [생략] | 신규 QA 신설(정의+KPI 5축) + ISO Security 앵커 + QAS-A | red-team 세트 구축 [생략]; 번호 재정렬(OI-8); DP 보안 tactic(OI-7) |
| **NQA-B Correctness** | 신설·권장 | [반영](신설) + eval [생략] | 신규 QA 신설(golden 정답률) + ISO Functional Correctness + QAS-B | golden 구축 [생략]; eval 서브시스템 신규 DP(OI-7); 번호(OI-8) |
| **NQA-C Cost-economy** | 신설·권장(Med) | [반영](신설) + 일부 [발표 서사] | 신규 QA 신설($/완료모델) + ISO Resource Utilization + QAS-C | baseline 추정 [발표 서사]; KPI 이동 닫기; 번호(OI-8) |

## 2. 지적별 처리 (closure)

### 교차분석 C1~C5
- **C1 측정 불가 KPI 군집**(placeholder·"최대화") → **[반영]**: QA-01 `완료모델≥N`·`활용률≥70%` 폐기, QA-07 `{사람−에이전트} 최대화` → 노드타입별 speedup(임계값화).
- **C2 정의↔KPI 불일치** → **[반영]**: QA-08 정의(E2E)↔KPI(전달5%) 불일치 복구(E2E latency·throughput 추가), QA-03 QA↔QAS SSoT 통일(위반0 편입).
- **C3 QA 경계 중복(taxonomy)** → **[반영]**: Performance 3분할 명문화(QA-01 throughput / QA-07 per-node / QA-08 E2E), QA-02(복구)↔QA-06(격리) 경계 altitude로 분리.
- **C4 agentic 고유 리스크(eval 하네스 등)** → **일부 [반영](KPI 정의) + 일부 [생략]/[발표 서사](eval 실행)**: runaway cap·trace 완전성·캐시우회 pass^k·품질 게이트 등 KPI는 원본 반영. 그것을 증명하는 red-team/golden/eval 하네스 **실행**은 [생략], "검증하도록 설계" 서사만 [발표 서사].
- **C5 누락 1급 QA** → **[반영](신설)**: NQA-A/B/C 파일 신설(정의+KPI+ISO 앵커+짝 QAS). 단 **정식 채택·우선순위 번호 재정렬은 [이월]**(사람 결정, OI-8).

### 일괄 [생략]
- **17-PoC orchestration 전체**(`_poc-plan.md`의 의존그래프·5~6주 일정·critical path) → **[생략]**: 동작 시스템이 없어 실측 PoC 0건 → 실행 관리물 무의미. 발표 1장("검증 전략 개요")으로만 축소 인용.
- **개별 eval 서브시스템 구축**(NQA-A red-team 50~100건·NQA-B golden 단계별 라벨·judge 인간일치 검증) → **[생략]**(실행 노동). 단 "아키텍처 박스"로는 모듈 다이어그램에 존치(설계 산출물 = OK).

### [거부]
- 없음. round-01 counsel 13개 권고의 글쓰기 부분은 거의 전부 [반영], 검증 노동만 [생략]/[발표 서사]로 분기(자가당착·과에포트로 인한 미채택 없음).

## 3. 다음 Reviewer가 다시 볼 것 (재검증 요청)

이번 라운드 N+1의 **우선 점검 목록**이다.

1. **[발표 서사]로 미룬 검증이 여전히 유효한 설계인가** — QA-03 적대적 eval(위반0), ~~QA-09 캐시우회 pass^k~~, NQA-A red-team, NQA-B golden, NQA-C 수작업 baseline. "실측 없이 설계 서사만"이 발표 방어선으로 충분한지 재판정.
   - **갱신 2026-06-24**: **QA-09는 contention으로 해소** — 헤드라인을 시연 불가한 `pass^k 절대값`에서 시연 가능한 `캐시 On/Off 갭 Δ 대조`로 교체, ②를 ②-1 대리 게이트(반영)/②-2(open-issue)로 분리. 상세 [contention/counter.md](../contention/counter.md). 나머지(QA-03·NQA-*)는 미해결.
2. **교차 의존이 닫혔는가** — QA-07 first-pass 게이트·QA-09 유효-결정률이 **NQA-B에 의존**. NQA-B가 [이월](미채택)이면 두 QA의 KPI는 아직 안 닫힘 → 동반 채택 필요성 재확인.
3. **DP 역검토(OI-7)** — round-01 반영이 드러낸 DP 미명시 차원(rate-limit·외부 LLM degradation·runaway cap·안전정지·span trace·품질게이트·workflow 버저닝·보안 tactic·eval 서브시스템 신규 DP). **다음은 DP 디스커션 차례**임을 시사.
4. **신규 QA 정식화(OI-8)** — NQA-A/B/C 채택 가부 + 우선순위 번호 재정렬(Security·Correctness 상위 진입) → 확정 시 기존 QA cross-ref·INDEX·glossary 동기화.
5. **예시값의 현실성** — 0.8·99.5%·30초·6k·3배·6시간·pass^k 70%·CIS p95 3개·$5/모델 등은 전부 "측정가능 KPI의 모양" 예시. 발표 전 팀 합의 필요(레퍼런스 복제 아님).

## 4. 사람 결정 보류

- **주 KPI 선택** — QA-09는 주 KPI(pass^k)의 검증이 [발표 서사]라, "만들 PoC"가 아닌 "방법론 시연"으로 둠. 이 절충이 맞는지.
- **importance 상향** — QA-08(E2E top-line)·QA-09(자율신뢰 토대, NQA-B와 묶이면) 상향 여지(현재 M/L 유지).
- **NQA 정식 채택 + 번호 재정렬** — OI-8. 임시 ID(NQA-A/B/C) 유지 중.
- **예시값 전반** — 실환경 측정/모델로 확정.

---
> 출처 추적: 각 QA의 `context/qa/<ID>.md` ## 변경 이력 + `context/open-issues.md` OI-7/OI-8. 반영 커밋: `2f48e0c`(QA-01)·`650bba3`(QA-02)·`c5702d8`(QA-03~10)·`630b3e1`(NQA-A/B/C).
