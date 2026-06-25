# QA 리뷰 round-04 — 결론 리포트 (★ 등급 척도 red-team 검증)

> 이 문서 하나로 round-04 전모를 파악한다. 개별 근거는 QA-0X-*.md, 방법론은 ../../Reviewer.md.
> date: 2026-06-25 · scope: context/qa/ QA 13개의 ★ 등급 척도(round-03 캘리브레이션) red-team 검증 + 일반 4축
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 직전: round-02 review → round-02 applier → round-03(reviewer-less ★ 캘리브레이션) counsel
> 절차 0: round-03엔 applier report 없음(reviewer-less) → round-02 applier §3 + round-03 counsel.md(★ 설계 의도)를 입력으로 ★ 급간을 red-team 검증.

## TL;DR
- **★ 급간 재배치의 방향·구조는 옳다.** 13개 전부 규칙5(헤드라인=게이트/별점=gradable proxy 분리)를 **가짜 급간 날조 없이** 통과하고, OI-9 정합도 13개 전부 통과(§측정·등급표·변경이력·counsel·짝 QAS 일치, glossary 충돌 0). round-03이 잡은 비현실 하한 진단(QA-06 95→70, QA-11 70→25/40/60, QA-13 70→50, QA-01 0.8→0.70)과 보수 하한 진단(QA-10 6h→24h ★☆☆ 사문화 교정)은 정확. **신규 결함 0·Sound ✕ 0·KPI ✕ 0.**
- **red-team 핵심 = 근거 품질 4축.** C1 apples-to-apples 미검증(인용 벤치의 측정 셋업이 우리 QA와 다름), C2 PoC margin 미규칙화(경계마다 다른 임의 margin), C3 main 별점 축이 [발표 서사]라 실측 불가(QA-07·11·13 → ★가 모델 추정), C4 근거 URL 신뢰성(미래형 arXiv ID·단일 출처·블로그 — 웹 도구 부재로 확인 불가).
- **가장 날카로운 단일 지적: QA-11 ★★★ 60% 사문화 의심**(frontier pass^5 0.8^5≈33% < 60%, voting 근거 부재).
- **잔여 Med 3건(QA-07·11·13)** = 전부 C3(main 축 실측 불가). 나머지 10건 Low.
- **인용 정확성은 본 라운드 확인 불가(웹 도구 없음)** → 1차 출처 전수 재확인을 round-05 Council에 위임.

## 종합 판정표 (round-03 ★ 신설 → round-04 검증)

| QA | 속성 | R02 Sound/KPI·sev | R04 Sound/KPI·sev | ★ 급간 검증 핵심 |
|---|---|---|---|---|
| QA-01 | Scalability | ○/○ Low | ◎/○ Low | 0.8→0.70 USL 정합. ★☆☆ 대역(0.70~0.75) 좁음·margin 0.02~0.05 흔들림·SPARCcenter CPU USL apples |
| QA-02 | Availability | ○/○ Low | ◎/○ Low | 1분→4분 게이트/별점 분리 모범. ★★★ 10초 "재스케줄 0.5초 vs 재기동 10초" 단위 혼동·외부 outage 길이 미반영 |
| QA-03 | Controllability | ◎/△ Med | ◎/○ **Low** | 30초 흡수 = 규칙2·4·5 동시 모범. KPI △→○. ★★★ 15초 단일출처(k8s 30초) |
| QA-04 | Observability | ○/○ Low | ◎/○ Low | ★★★ 100%미만 캡 = CoT 비공개 정직. ★★☆ 대역(95~98) 좁음·경계표기 모호 |
| QA-05 | Efficiency | ○/○ Low | ◎/○ Low | tier(4/6/8k) 정식화. 적중률 ≥85% 조건이 ★ 변별 폭 가림·Requesty 단일출처 |
| QA-06 | Security | ○/△ Med | ◎/○ **Med** | 95→70 정합·규칙5 정확. 직접/간접 injection apples 미검증·★★☆ 80경계 margin 11%p 불일치·FPR게이트 ★★★만 |
| QA-07 | Correctness | ○/△ High | ◎/○ **Med** | 만점 캡(92~98)+κ 조건 모범. KPI △→○. main축 실측불가·judge κ 도메인 의존·★★☆(90~92) 좁음·(98,100) 공백 |
| QA-08 | Reliability-WF | ○/○ Low | ◎/○ Low | 10→25% 무격리바닥 간격. ★★★ 5% margin 근거 약함·쿼터(진짜 공유장애)는 ★에 미반영·미래형 URL |
| QA-09 | Perf per-node | ○/△ Med | ◎/○ **Med** | 3배→1× 협업/완전자율 대비. KPI △→○. ★★★ 8배 InfEngine 단일+미래형 URL·하한표현 혼란·QA-07 게이트 의존 |
| QA-10 | Perf E2E | ○/○ Low | ◎/○ Low | 6h→24h = ★☆☆ 사문화 교정(세트 유일). ★★★ 2h margin 약함·agentic LLM 큐잉 가산 미명시 |
| QA-11 | Reliability-일관성 | ○/△ Med | ◎/○ **Med** | 70→25/40/60 정합·규칙5 정확. KPI △→○. **★★★ 60% 사문화 의심(0.8^5≈33%)**·main축 실측불가·25하한 k=8 보간·§측정 구값 잔존 |
| QA-12 | Maintainability | ○/○ Low | ◎/○ Low | CIS p95 재배치 불요. 정수 입도(1/2/3) 거침·경계정의 종속·agentic 축이 보조게이트로 밀림 |
| QA-13 | Cost-economy | ○/△ Med | ◎/○ **Med** | 70→50% = 규칙2 핵심. KPI △→○. ★ 전경계 baseline 추정([발표서사]) 좌우·★★★ 75% 법무 단일사례·OI 미등록 의심 |

집계(13): **Sound ✕ 0 · KPI ✕ 0 · High 0 · Med 3(QA-07·11·13) · Low 10.** KPI △→○ 5건(QA-06·07·09·11·13). 기존 QA 10만: R02 High 0·Med 3·Low 7 → R04 Med 1(QA-09)·Low 9 + 신설편입 QA-06/07/13. 신규 결함 0.

## 교차분석 C* — round-04

### C1 — ★ 경계의 apples-to-apples 미검증 (전 세트)
인용 필드 벤치마크의 측정 셋업이 우리 QA와 다른데 ## 등급 척도 노트에 미명시: QA-06(Unit 42 직접 injection·단발 prompt vs 우리 agentic 간접 injection), QA-11(τ-bench retail/airline 고객응대 vs SDK 빌드 파이프라인), QA-05(Requesty 코딩 에이전트 vs 빌드 노드), QA-07(일반 벤치 judge κ vs quantize 도메인 judge), QA-09(InfEngine assistant-type vs SDK 노드 종류), QA-13(법무 멀티에이전트 절감 vs 빌드 절감), QA-01(SPARCcenter CPU USL vs LLM-bound worker), QA-08(스토리지/CPU noisy-neighbor vs 토큰 쿼터). → round-05 Council이 각 경계의 셋업 차이를 silent cap으로 명시하거나 우리 PoC 재측정 단서 보강.

### C2 — PoC margin이 규칙화되지 않은 임의값 (다수)
규칙3 "이론천장 − PoC margin"의 margin이 경계마다 다른 정량 근거 없는 값: QA-06(★★★ 5%p vs ★★☆ 11%p), QA-01(★★★ 0.05 vs ★☆☆ 0.02), QA-08(★★★ 5% 근거 없음), QA-10(★★★ 2h="6h의 1/3" 근거 없음), QA-13(★★☆ 15%p vs ★☆☆ 10%p), QA-05(2k 등간격 margin 미명시). → margin 산정 규칙(예: 일괄 "이론/필드천장 − Y%p"의 Y를 QA별 1개로 고정) 권고.

### C3 — main 별점 축이 [발표 서사]/실측 불가인 군집 (신규·구조)
별점을 매기는 main 축이 우리 조건에서 실측 불가 → ★가 모델 추정에 그쳐 ATAM 변별력 약화: QA-11(pass^5 절대값), QA-07(golden 정답률 — golden·judge 미구축), QA-13(절감률 — 수작업 baseline 추정 의존). 세 QA 모두 정직하게 silent cap으로 자인하나, **산출 가능한 보조지표(QA-11 ②-1/H_norm, QA-07 rework율, QA-13 $/모델 분해)를 보조 별점 축으로 병기**해 "모델 추정(main) + 실측 가능(보조)" 이중 축으로 변별력 보강 권고. (QA-03·06은 별점 축이 [생략]됐어도 원리상 측정 가능 — 비대칭.)

### C4 — 근거 출처 신뢰성 (신규·중요)
다수 ★ 경계가 (a) 미래형 arXiv ID(2602.18985·2604.03145·2511.07413·2511.15759·2504.11168 — 26xx=2026년)·(b) 단일 출처·(c) 블로그/벤더 자료에 의존. 본 라운드는 웹 도구 부재로 실재·정확 인용을 확인 불가. → round-05 Council이 13개 QA의 인용 수치를 1차 출처로 전수 검증(특히 Unit 42 53/91/92%·FPR 13.1%, τ-bench pass^1 80/61%·pass^4 37%·pass^8 25%, InfEngine 8.6~22.7×, 미래형 ID 실재 여부).

### C 추적 (round-02 → round-04)
round-02 C1(QA-07·09↔NQA-B 교차의존)은 OI-8 정식 편입(QA-07)으로 닫힘 경로 생김 — 단 QA-07 golden 미구축([발표 서사])이라 QA-09 first-pass 게이트·QA-11 ②-2는 여전히 QA-07에 의존(C3와 연결). DP 귀속 placeholder(OI-7)는 DP 디스커션 미착수로 잔존.

## 신규 QA 후보 (NQA) — round-04
**없음.** taxonomy 공백은 OI-8(QA-06·07·13 편입)로 닫혔고, ★ 급간 검증에서 새 1급 QA는 식별되지 않음. 상세 _new-qa-candidates.md.

## 직전 라운드 대비 변화 (round-03 → round-04)
- round-03은 reviewer-less(팀이 reviewer, Council 3 seats가 ★ 캘리브레이션) → round-04가 그 ★ 급간을 **처음 red-team 검증**.
- ★ 신설로 **KPI △→○ 5건**(QA-06·07·09·11·13). 잔여 Med 3(QA-07·11·13)은 전부 C3(main 축 실측불가).
- OI-9(하한 보정 ↔ QAS·glossary 정합) **13개 전부 통과.** 미세 잔존 2건: QA-11 §측정 캐시노트 구값 `pass^k 70%`, QA-13 변경이력 "OI 등록 범위 외"(QA-08~10은 등록 명시 — QA-13 미등록 의심).
- 다음(round-05 Council): C1~C4 근거 보강 + C3 보조 별점 축 검토 + QA-11 ★★★ 60% 도달성 재판정.

---
> 메타: scope=full(샘플 QA-06·QA-11 + 나머지 11 완성). 날짜 2026-06-25. red team 산출물 = round-04/review/. blue team(Council)·반영(Applier)은 후속 단계.
