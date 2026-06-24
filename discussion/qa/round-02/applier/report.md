# Applier 반영 보고서 — round-02 (concept=qa)

> 반영 대상: round-02의 [review](../review/report.md) + [counsel](../counsel/counsel.md) + [contention](../contention/counter.md)(QA-03 ASR)
> 반영 일자: 2026-06-24 · 반영 범위: QA-03(contention 델타) + Low cross-link 4건(QA-01·05·06·10) · 원본: `context/qa/`
> 다음 라운드 Reviewer는 이 보고서를 입력으로 읽고, 각 지적의 처리를 확인해 verdict 변화를 재평가한다.
> 입력 우선순위: **contention counter 델타 > round-02 counsel > review**. round-02엔 `_feasibility-filter.md`가 없음(수렴 라운드) — counsel.md §종료조건 평가의 "actionable [반영]거리 = QA-03 contention + Low cross-link"를 등급 기준으로 적용.

## 1. 한눈에 (항목별 처리 요약)

| ID | review verdict (R02) | 처리 등급 | 무엇을 바꿨나(1줄) | 이월/재검증 |
|---|---|---|---|---|
| QA-03 Controllability | ◎ / △ · Med | **[반영]**(contention) + [이월] | ② 위반0 → ②-1 차단율(현 조건 닫힘)/②-2 acceptance(open-issue) 2단 분리 + "동반 닫힘" 조건부 정정 + 헤드라인 ① 유지 | ②-2 ↔ NQA-A 채택(OI-8); cap/eval DP(OI-7) |
| QA-01 Scalability | ○ / ○ · Low | **[반영]**(Low) | rate-limit 헤드룸 측정단위(계정전역 TPM/RPM·큐별 token-bucket·QA-06 cross-link) | 활용률 NQA-C(OI-8); rate-limit DP(OI-7) |
| QA-05 Efficiency | ○ / ○ · Low | **[반영]**(Low) | 캐시 적중률 보조 KPI 노출(측정 정직성) | top-line NQA-C(OI-8); 비용 라우팅 DP(OI-7) |
| QA-06 Reliability-WF | ○ / ○ · Low | **[반영]**(Low) | 외부 rate-limit 계정전역 silent cap 명문화(QA-01 cross-link) | 격리 DP-0004/0005(OI-7) |
| QA-10 Maintainability | ○ / ○ · Low | **[반영]**(Low) | CIS p95 컴포넌트 경계 정의 선결 명문화 | 교체내성 DP(OI-7) |
| QA-02 Availability | ○ / ○ · Low | [생략](닫힘 재확인) | 신규 반영 없음(round-01 4축 무손실 반영분 유효) | 외부 LLM degradation DP·`drives` 교정(OI-7) |
| QA-04 Observability | ○ / ○ · Low | [생략](닫힘 재확인) | 신규 반영 없음(span 환원 반영분 유효) | DP-0003 span 보장(OI-7) |
| QA-08 Performance-E2E | ○ / ○ · Low | [생략](닫힘 재확인) | 신규 반영 없음(mislabel 복구 유효) | 5% 산식 실측·importance 상향(사람) |
| QA-07 Performance-Agent | ○ / △ · Med | [이월] | 주 KPI가 NQA-B golden 게이트 의존 — 자체 수정 불가 | NQA-B 채택(OI-8·세트 병목) |
| QA-09 Reliability-일관성 | ○ / △ · Med | [이월] | contention(round-01) 부분 닫힘 유지, ②-2만 NQA-B 의존 | NQA-B 채택(OI-8); importance(사람) |
| NQA-A Security/Safety | ○ / △ · Med | [이월] | 정식 채택·번호 = 사람 결정 | OI-8; 보안 tactic DP(OI-7) |
| NQA-B Correctness | ○ / △ · High | [이월] | 정식 채택 = 세트 닫힘 단일 트리거·사람 결정 | OI-8 최우선; eval DP(OI-7) |
| NQA-C Cost-economy | ○ / △ · Med | [이월] | 정식 채택 = QA-01/05 부유 해소·사람 결정 | OI-8 |

집계: **new [반영] 5건**(QA-03 contention 1 + Low cross-link 4) · **[생략](닫힘 재확인) 3건**(QA-02·04·08) · **[이월] 5건**(QA-07·09·NQA-A·B·C). [거부] 0건(QA-03 R2 헤드라인 교체는 [거부·정정] — §2 참조).

## 2. 지적별 처리 (closure)

### QA-03 contention (최우선 — counter 델타 > counsel)
- **R1 [수용] — "동반 닫힘" 조건부성 은폐** → **[반영]**: ②를 ②-1(권한외 차단율·룰 체커·admission 게이트, golden/하네스 풀세트 불요)과 ②-2(적대적 위반0 acceptance·NQA-A 풀커버리지 의존)로 2단 분리. ②-1은 OI-8 무관 **현 조건 닫힘 1차 게이트**로 원본 KPI에 기입. "동반 닫힘" 무조건 표현 → "②-2·NQA-A는 OI-8 채택 시 동반 닫힘; ②-1은 선닫힘" 조건부로 KPI·검증 전략에 정정.
- **R2 [부분수용] — 절대값 시연 불가 + QA-09 비대칭** → **[반영](수용 부분) + [거부·정정](헤드라인 교체)**: ②-1 측정을 "주입 위반 → 차단/안전정지 latency 분포"로 통일(①의 cancel→ack/정지 분포와 동일 아키타입 — "위반 부재"가 아니라 "차단 메커니즘 작동" 시연). red가 요구한 "헤드라인 교체"는 거부·정정(아래 [거부] 참조).
- **②-2 "0건" 커버리지 silent cap** → **[반영](기존 유지)**: OWASP LLM Top-10 매핑·세트 커버리지=신뢰 상한을 KPI·검증 전략에 유지(과대주장 방지, contention §해소).
- **DP 귀속(runaway cap·graceful stop = OI-7)** → **[이월]**: contention 범위 밖·DP 디스커션 위임(rebuttal에서도 해소 판정).
- **②-1 latency를 ① PoC에 합칠지 별도 마이크로-PoC인지** → **[이월·재량]**: PoC 밀도 문제(측정가능성·정합성 무관), Applier/사람 재량.

### Low cross-link 4건 (counsel이 actionable [반영]거리로 든 것)
- **QA-01 rate-limit 헤드룸 측정단위** → **[반영]**: 계정 전역 TPM/RPM 한도 대비 헤드룸 + 큐별 token-bucket 쿼터 배분, WF별 쿼터 격리는 QA-06 담당(token-bucket 인프라 공유) cross-link.
- **QA-05 캐시 적중률 보조 KPI** → **[반영]**: 주 KPI(신규 토큰 ≤6k) 합격이 적중률에 좌우되므로 측정 정직성 차원에서 보조 노출(워크로드 다양성 의존 silent cap 정합).
- **QA-06 외부 rate-limit 계정전역 silent cap** → **[반영]**: ③ 쿼터 침범 KPI 옆에 token-bucket=client-side 분배만 보장(제공자측 공유 한도 못 늘림)·QA-01 헤드룸과 같은 천장 cross-link.
- **QA-10 컴포넌트 경계 정의 전제** → **[반영]**: CIS p95 KPI 옆에 컴포넌트 경계 정의 선결(측정 전제·silent cap) 명문화.

### 닫힘 재확인 7건 (verdict 닫힘은 round-01 반영분)
- **QA-01·02·04·05·06·08·10**: review report 종합 판정표에서 KPI ○로 닫힘 확인. 본문 verdict 닫힘 자체는 round-01에 이미 반영됨 → round-02엔 위 Low 보강(QA-01·05·06·10) 외 **본문 대수정 없음**. QA-02·04·08은 신규 반영 없음(닫힘 재확인만).

### 이월 (자동 반영 아님 — 전제 미충족)
- **NQA-A/B/C 정식 채택·우선순위 번호 재정렬** → **[이월]**: OI-8 사람 결정. 임시 ID(NQA-A/B/C) 유지. 채택 시 QA-01~10 재번호·INDEX/glossary/cross-ref 동기화 필요.
- **eval/검증 서브시스템 DP 신설·KPI-DP 귀속** → **[이월]**: OI-7 DP 디스커션(C3 수렴 = 단일 eval/검증 DP·1순위 의제). QA 측 verdict는 내려가지 않음.
- **QA-07 주 KPI·QA-09 ②-2 동반 닫힘** → **[이월]**: NQA-B 채택(OI-8)에 묶여 미닫힘. QA-03 ②-2 동반 닫힘은 NQA-A 채택(OI-8)에 묶임.
- **예시값 확정**(5초·30초·0.8·20%·6k·1%·10%·CIS p95 3개 등) → **[이월]**: 팀 합의·실측/모델로 확정(레퍼런스 복제 아님).

### [거부]
- **QA-03 "헤드라인 ②로 교체"**(red rebuttal R2 한 축) → **[거부·정정]**. 사유: QA-03 헤드라인(주 KPI·PoC 대상)은 ①(ack/안전정지 latency, 시연 가능)이고 ②는 처음부터 보조 KPI라 "교체" 대상이 아님 — counter가 KPI 지위 차이를 정당한 비대칭 근거로 제시, referee 불요. "절대값→대리/대조" 원칙 자체는 ②-1 분리로 동일하게 적용됨(원칙 거부 아님, 적용 지점만 보조 KPI로 정정).

## 3. 다음 Reviewer가 다시 볼 것 (재검증 요청)

round-03(또는 NQA 채택 후 검증 라운드)의 **우선 점검 목록**.

1. **QA-03 ②-1이 verdict를 움직였는가** — ②-1(룰 체커 차단율, OI-8 무관 선닫힘)이 KPI에 기입됨. QA-09가 ②-1로 부분 닫힘(△ 유지)이 된 것과 같은 구조 — QA-03도 ②-1 반영 후 KPI △가 "부분 닫힘 △"로 재평가되는지(②-2만 잔존 의존). 닫힘도는 QA-03 > 단순 △ 기대.
2. **②-2 ↔ NQA-A 교차 의존(OI-8)** — QA-03 ②-2 적대적 위반0은 NQA-A 공유 red-team 하네스 채택에 의존(open-issues OI-7·OI-8 등록 완료). NQA-A 채택이 QA-03 ②-2와 NQA-A 주 KPI를 동반 닫는 단일 행동인지 재확인. (QA-09 ②-2 ↔ NQA-B와 동일 구조.)
3. **Low 보강 4건이 verdict에 영향 없는지** — QA-01·05·06·10은 이미 KPI ○ Low. cross-link/단위 명문화가 닫힘을 흔들지 않고 보강만 했는지 확인(verdict 변화 0 기대).
4. **[이월] 5건은 QA 라운드가 아니라 다른 디스커션·사람으로** — NQA-A/B/C 채택(OI-8)·QA-07/09 동반 닫힘은 QA 라운드 N+1로 해결 불가. eval/검증 서브시스템 DP(OI-7)는 DP 디스커션 1순위.
5. **예시값 현실성** — 위 예시값 전부 "측정가능 KPI의 모양". 발표 전 팀 합의.

## 4. 사람 결정 보류

- **NQA 정식 채택 + 번호 재정렬(OI-8)** — 최우선. 우선순위 권고: NQA-B(세트 병목·High) → NQA-A(QA-03 ②-2 공유·상위 진입) → NQA-C(QA-01/05 부유 해소·Med). 채택 시 재번호·양방향 cross-link·INDEX/glossary 동기화.
- **DP 디스커션 착수(OI-7)** — eval/검증 서브시스템 단일 DP 신설(C3 수렴) + QA별 DP 귀속(runaway cap·graceful stop·rate-limit headroom·비용 라우팅·교체내성·외부 LLM degradation·DP-0001 `drives:QA-02→QA-06` 교정).
- **importance 상향 여지** — QA-08(E2E top-line)·QA-09(NQA-B 묶이면, contention 게이트 충족)·NQA-A/B(신뢰 두 기둥). verdict 무관 우선순위.
- **예시값 전반** — 실환경 측정/모델/baseline로 확정.
- **수치 자가당착 점검(Applier 발견)** — 발견된 자가당착 **없음**. QA-03 ②-1/②-2 분리는 측정 모순 없이 닫힘(②-1 현 조건·②-2 채택 의존으로 깔끔히 갈림). 단 ②-1 "차단율" 합격선 수치는 예시값 미지정(0건 절대값이 아니라 분포라 placeholder ◯ 대신 "주입 N건 대비 차단율 + latency 분포"로 *모양*만 기입) — 합격 임계는 사람 확정.

---
> 출처 추적: 각 QA `## 변경 이력`(2026-06-24 round-02 블록) + `context/open-issues.md` OI-7/OI-8 + `context/changelog.md`(2026-06-24 round-02 델타). 반영 대상 contention: [counter.md](../contention/counter.md) R1[수용]·R2[부분수용]. 커밋은 호출자/사람이.
