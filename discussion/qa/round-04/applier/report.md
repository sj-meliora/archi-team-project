# Applier 반영 보고서 — round-04 (concept=qa)

> 반영 대상: round-04의 review + counsel (★ 등급 척도 근거 보강 라운드)
> 반영 일자: 2026-06-25 · 반영 범위: QA-01~13 전체(델타 있는 것) + 짝 QAS · 원본: context/qa/
> 다음 라운드 Reviewer는 이 보고서를 입력으로 읽고, 각 지적의 처리를 확인해 verdict 변화를 재평가한다.
> 이 라운드 특수 사정: _feasibility-filter.md 없음(수렴/캘리브레이션 라운드) → counsel.md 채택 권고표를 등급 기준으로 삼음. contention 없음(counter.md 입력 없음).

## 1. 한눈에 (항목별 처리 요약)

| ID | review verdict (R04) | 처리 등급 | 무엇을 바꿨나(1줄) | 이월/재검증 |
|---|---|---|---|---|
| QA-11 Reliability-일관성 | ◎/○ · Med | [반영](급간 수치) | ★★★ pass^5 60→45%(★★☆ 35~45) + ②-1 보조 별점 축 + k 보간/apples silent cap + §측정 구값 정정 | pass^5 절대값 실측·②-2↔QA-07(OI-7) |
| QA-06 Security | ◎/○ · Med | [반영](급간 수치) | ★★☆ 80~90→85~90%(margin 5%p)·FPR ≤1% 전 급간 게이트·직접/간접 injection apples 1차출처·세트 표본오차 cap | red-team 세트(발표 서사)·DP(OI-7) |
| QA-07 Correctness | ◎/○ · Med | [반영](급간 수치) | ★★★ 92~98→[93,99]·★★☆ [91,93)·100%만 불합격·judge κ 도메인·rework율 보조축 | golden·judge(발표 서사)·OI-7 |
| QA-09 Perf-Agent | ◎/○ · Med | [반영](인용 정정) | InfEngine 정정(21×·92.7%·적외선; 8.6~22.7×·assistant는 오류)·apples·하한 표현 단일화·tool 시간 분리. ★★★ ≥8배 유지 | first-pass↔QA-07(OI-7) |
| QA-04 Observability | ◎/○ · Low | [반영](표기·정의) | 경계 [95,97)/[97,98)/[98,100) 구간화·완전성=존재 AND 비-truncation. 하한 불변 | DP-0003 span(OI-7) |
| QA-12 Maintainability | ◎/○ · Low | [반영](표기·보조축) | CIS ≤1/1<p95≤2/2<p95≤3 구간화·prompt/tool 보조축·경계 종속 cap. 하한 불변 | DP-0004/0005·FR-0003(OI-7) |
| QA-13 Cost-economy | ◎/○ · Med | [반영](보조축·근거) | baseline 추정 cap·재시도 보조축·★★★ 75% 도메인무관 재근거·OI-9 등록 확인 해소. 급간 수치 불변 | baseline(발표 서사)·DP(OI-7) |
| QA-01 Scalability | ◎/○ · Low | [반영](근거) | margin 천장거리 비례·LLM-bound USL apples cap. 급간 수치 불변 | rate-limit DP(OI-7) |
| QA-08 Reliability-WF | ◎/○ · Low | [반영](근거) | ★★★ ≤5% margin(tail+폭주)·arXiv 미래형 검증결과·쿼터 0건 Constraint 유지. 급간 수치 불변 | DP-0005 ④↔R-1(OI-7) |
| QA-10 Perf-E2E | ◎/○ · Low | [반영](근거) | ★★★ 2h=compute critical path 하한·agentic E2E=배치 ML+LLM 큐잉. 급간 수치 불변 | DP-0004 산식(OI-7) |
| QA-02 Availability | ◎/○ · Low | [반영](정의·근거) | "재기동"=lease 만료→체크포인트 재개 정의 고정·외부 outage cap·lease trade-off. 급간 수치 불변 | DP-0002/0003 degradation(OI-7) |
| QA-03 Controllability | ◎/○ · Low | [반영](근거) | ★★★ 15초 근거 다양화(k8s 이등분+Temporal)·롤백 외부 정합성 한 줄. 급간 수치 불변 | runaway cap DP(OI-7) |
| QA-05 Efficiency | ◎/○ · Low | [반영](근거) | 적중률 조건 cap·tier=난이도 매핑·Requesty 단일출처 다양화. 급간 수치 불변 | DP-0001 비용 라우팅(OI-7) |

집계: new [반영] 13건. [거부] 0 · [이월] 0. C3 보조 별점 축 4개(QA-07·11·12·13) 신설. 잔여는 OI-7·발표 서사 기존 트래킹.

## 2. 지적별 처리 (closure)

### C1 apples-to-apples → [반영]
각 ★ 노트 silent cap: QA-06(Unit 42=단발 직접 injection vs 우리 간접 — 1차출처 확인) / QA-01(SPARCcenter CPU USL) / QA-05(Requesty 코딩) / QA-07(일반 judge κ) / QA-08(스토리지 noisy-neighbor) / QA-09(적외선 컴퓨팅) / QA-13(법무). QA-03·04·10·12는 우려 낮아 유지.

### C2 margin 미규칙화 → [반영]
QA-06 단일 Y=5%p / QA-01 천장거리 비례(0.05/0.02) / QA-08 tail+폭주 흡수 ≈5% / QA-10 compute critical path 2h / QA-13 분포 두께 차등 / QA-05 tier=난이도 매핑.

### C3 main 축 실측불가 → [반영](이중 축 신설)
QA-11 ②-1(95/97/99%)·QA-07 rework율(≤5/15/30%)·QA-12 prompt/tool 수정 수(≤1/=2/=3)·QA-13 재시도 오버헤드(≤10/25/40%). QA-08 쿼터는 0건 절대형이라 별점화 불가 → Constraint 유지.

### C4 출처 신뢰성 → [반영]
QA-09 InfEngine 인용 정정(가공 수치). QA-11 voting→pass^k 메커니즘 반증으로 ★★★ 60→45%. QA-06 Unit 42·τ-bench 정확 확인. QA-08 arXiv 2604.03145 "확인 불가" 정직 표기.

### 개별 핵심
QA-11 ★★★ 60% 사문화(최날카) → [반영] 45% 하향 / QA-07 (98,100) 공백 → [반영] / QA-04 경계 모호 → [반영] 구간화 / QA-13 OI 미등록 의심 → [반영] OI-9 등록 해소.

## 3. 다음 Reviewer가 다시 볼 것
1. QA-11 ★★★ 45%가 verdict를 움직였는가(voting 반증 수용 여부, ②-1 보조축 실측 변별).
2. C3 이중 축 4개가 ATAM 변별력을 살렸는가(main 동률 시 보조축 변별).
3. QA-09 InfEngine 정정 후 ★★★ ≥8배 근거(21× 단일·도메인 불일치 잔존).
4. [발표 서사] 미룬 검증: QA-06 red-team 세트·QA-07 golden+judge·QA-13 baseline·QA-11 pass^5 절대값.
5. OI-7(DP 디스커션) 잔존 — QA 라운드로 해결 불가, DP 1순위.
6. 예시값 현실성(45%·85%·[93,99]·재시도 10/25/40% 등) 발표 전 팀 합의.

## 4. 사람 결정 보류
- 수치 자가당착 — 없음. OI-9 5곳 일관, glossary 충돌 0(수치 미포함). 단 QA-09 ★★★ ≥8배는 21× 단일·도메인 불일치 — "단일 출처" 약점 잔존, 경계 유지 여부 PoC/사람 확정 권고.
- importance 상향 여지(QA-06·07·08·09·13)·DP 디스커션 착수(OI-7)·예시값 전반 팀 합의.
- 번호 재정렬·NQA 신설 — 본 라운드 해당 없음(신규 0·재번호 0).

---
> 출처 추적: 각 QA ## 변경 이력(2026-06-25 round-04 블록) + context/open-issues.md OI-9 round-04 항목 + context/changelog.md(2026-06-25 round-04 델타). 반영 기준: round-04 counsel.md 채택 권고표(filter 없는 수렴 라운드). 커밋은 호출자/사람이.
