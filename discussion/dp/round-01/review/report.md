# DP 디스커션 round-01 — review 종합 보고서 (red team)

> concept: dp · round: round-01 · scope 현재: **sample (골격 + 대표 2개)** · 날짜: **2026-06-25**
> 대상 스냅샷: context/dp/ DP-0001~0005 (+ dp4/ 하위) · ASR = QA-01~07 · prior art: dp4/review/findings.md (F-01~F-12)
> ⚠️ **이 보고서는 골격이다.** Traceability Matrix(필수·완성)·orphan ASR은 확정. DP-01/03/05 상세·교차발견(C*) 본문·NDP 확정은 **full 단계**에서 채운다. 현재 판정표는 샘플 2개(DP-02·DP-04)만 채움.

---

## 🔑 Traceability Matrix — DP × ASR (필수 · 렌즈2 산출) *(완성)*

> 행=DP, 열=ASR(QA-01~07), 셀=S/T/R/N (해당 DP 문서 ATAM 4절과 1:1, grep 역참조). 빈칸=무관. **열 전체 빈칸=orphan ASR.**

| DP \ ASR | QA-01 Scal | QA-02 Avail | QA-03 Ctrl | QA-04 Obs | QA-05 Eff | **QA-06 Sec** | **QA-07 Corr** |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **DP-01** WF–Agent 매핑 | **S,R** | N⚠️ | | | N | | |
| **DP-02** Agent Hierarchy | N | **S,T,R** | **S,T,R** | | (R)¹ | | |
| **DP-03** 외부 시스템 안정성 | | **S,T,R** | **S,T** | **S,T** | S,N | | |
| **DP-04** WF 실행 구조 | N² | | | | | | |
| **DP-05** E2E 캐시 | | | | | | | |
| **열 cover?** | ✅(DP-01 S) | ✅(DP-02/03 S) | ✅(DP-02/03 S) | ⚠️단일(DP-03) | ✅(DP-01/03) | **❌ orphan** | **❌ orphan** |

¹ DP-02 R-2 "2안 분산정책 → 토큰 비용 증가(QA-05)"로 약하게 닿음 — driving 헤더엔 QA-05 없음(약한 R).
² DP-04 유일 ASR. A5·A8 둘 다 Scalability ★★★ 동률 → cover하나 **변별 0**(N으로 표기, R/S 아님).

### 셀 근거 (grep 역참조 — DP 문서 ATAM 4절)
- **DP-01**: SP-2(정적/동적→QA-01 자원활용률≥70%)=S · R-1(1안 폭증 시 활용률 미달)=R → QA-01 **S,R**. QA-02: 헤더 `drives:QA-02`이나 ATAM에 QA-02 SP/TP/R 없음(NR-1만 generic) → **N⚠️**(OI-7: drives를 QA-02→QA-08 교정 검토 = 단방향 불일치). QA-05: Efficiency 강점(2안 ★★★)이나 ATAM 약함 → N.
- **DP-02**: SP-1(Orchestrator 가용성→QA-02)·R-1=R, TP-1(Ctrl↔Avail)=T → QA-02 **S,T,R**. SP-2(정책 집중→QA-03)·R-2=R, TP-1=T → QA-03 **S,T,R**. NR-1(Scalability 1·2안 동급)=QA-01 **N**.
- **DP-03**: SP-1(차단시점→QA-02)·R-1/R-2(C-0002 위반)=R, TP-1(Avail↔Perf)=T → QA-02 **S,T,R**. TP-2(예방력↔관측), 정책집행→QA-03 **S,T**. TP-2(→QA-04 Observability)·3안 강점 → QA-04 **S,T**. SP-2(탐지방식→QA-05 토큰)=S, NR-1(저오버헤드 안전)=N → QA-05 **S,N**.
- **DP-04**: driving 4개 중 ASR은 QA-01만. SP/TP/R 전부 QA-08/10/12(비-ASR)에 분포 → QA-01만 **N**(★★★ 동률, 변별 없음). 상세 → DP-04-*.md.
- **DP-05**: driving = QA-10·QA-08·QA-11·QA-12 — **전부 비-ASR**. ATAM SP/TP/R이 ASR(QA-01~07) 어느 칸도 채우지 않음 → **ASR 기여 0행**(세트 C* 후보).

### orphan ASR (열 전체 빈칸 = 최상위 결함)
| orphan ASR | 중요도 | 현 상태 | 메울 NDP |
|---|:---:|---|---|
| **QA-06 Security/Safety** | **H** | QA-06 `related-dp:[DP-0002,DP-0003]`이나 **두 DP 모두 `drives`에 QA-06 없음 + ATAM에 secrets/injection/공급망 SP/R 0** → 단방향 dangling 참조. OI-7: "DP-0002/0003 보안 tactic 미명시" 트래킹 중 | **NDP-A** (eval/검증·security gate 서브시스템) |
| **QA-07 Correctness** | **H** | QA-07 `related-dp:[]` 명시 공란 · 어느 DP도 안 다룸 · OI-7: "eval/검증 서브시스템 = 신규 DP 후보" | **NDP-A/B** (golden+judge 하네스 DP) |

> 두 orphan 모두 **H 중요도** + ASR(QA-01~07 상위) — **세트 최상위 결함**. QA-06/07은 round-01 QA 디스커션이 발굴해 정식 편입(OI-8)됐으나 **대응 DP가 미생성** → DP 세트가 ASR을 따라잡지 못한 상태. full 단계에서 NDP로 확정.
> 부차 경보: **QA-04 단일 DP cover**(DP-03만, R 없음 — 얇음) · **DP-05 ASR 기여 0행**(비-ASR 전용 DP).

---

## 판정표 (verdict 요약)

> 헤드라인 = `ASR / KPI` 두 1순위 축 + severity. 나머지 4축은 각 DP-0X-*.md 판정표. **현재 샘플 2개만 채움 — DP-01/03/05는 full 단계.**

| DP | ASR | KPI | severity | 한 줄 |
|---|:---:|:---:|:---:|---|
| **DP-02** Agent Hierarchy | ○ | △ | Med | ASR 3개(QA-01/02/03) 최다 cover·TP-1 실재 · ★칸 미앵커·3안 별점 추정·제어평면 DP-04 동시결정 미명시 |
| **DP-04** WF 실행 구조 | △ | △ | Med | A5 vs A8 수렴은 모범 · 단 ASR은 QA-01 1개뿐+동률 변별0 · 풍부함이 전부 비-ASR(QA-08/10/12)에 쏠림 |
| DP-01 WF–Agent 매핑 | — | — | — | *(full)* QA-01 S,R cover · drives QA-02 단방향 불일치(OI-7) 재판정 예정 |
| DP-03 외부 시스템 안정성 | — | — | — | *(full)* QA-02/03/04 다수 cover·조합(1+2안) 권고 · 보안 tactic 미명시(QA-06 orphan 기여) |
| DP-05 E2E 캐시 | — | — | — | *(full)* **ASR 기여 0**(비-ASR 전용) · 공유 캐시 장애전파(F-09 누적) |

**현재 severity 집계(샘플 2개)**: High 0 · Med 2 · Low 0. (orphan ASR 2건은 세트 결함으로 C*/NDP에 계상 — 개별 DP severity와 별도.)

---

## 교차발견 (C*) *(자리 — full 단계 본문화)*

> 세트 전체 구조 결함. 현재는 후보 목록만. full에서 §5 형식으로 본문화.

- **C1 (예정) orphan ASR — QA-06·QA-07 미cover** *[최상위]*: 위 Traceability Matrix 참조. H 중요도 ASR 2개가 어느 DP에도 없음 → NDP-A/B로. (severity High)
- **C2 (예정) 이중 제어평면 동시결정 누락 (F-03)**: DP-02(Orchestration/Choreography) ↔ DP-04 축 B(A3/A6)가 **동일 결정**인데 DP-02 본문이 동시결정성 미명시. evaluation.md(dp4)만 못박음 → 일방. (severity High 후보)
- **C3 (예정) 공유 장애도메인 누적 (F-09)**: DP-02 Orchestrator + DP-04 A8 로컬 디스크 + DP-05 공유 캐시 = 3개 공유 장애도메인 동시 도입. 합산 blast-radius vs QA-08 중단 KPI 미산정. (severity Med)
- **C4 (예정) 멱등 load-bearing 가정 ↔ QA-11 모순 (F-05)**: DP-04 재시도/재실행·DP-02 페일오버 재개·DP-05 캐시 재사용 모두 멱등 전제 — QA-11 비결정성(`pass^k`)과 충돌. 어느 DP도 SP/Risk 미승격. (severity Med)
- **C5 (예정) DP-05 ASR 기여 0 + drives↔ASR 정렬 부재**: DP-05 driving 전부 비-ASR. DP-01 drives QA-02 단방향 불일치(OI-7). DP-04 driving 4개 중 ASR 1개. → 세트의 drives 선언이 ASR 세트와 체계적으로 어긋남. (severity Med)
- **C6 (예정) ★칸 KPI 미앵커 — 세트 공통**: DP-01~05 ★매트릭스 칸이 round-03/04로 캘리브레이션된 QA 현 KPI(scaling efficiency≥0.70·재기동≤4분·graceful≤30초 등)에 미앵커 — DP는 옛 KPI 인용. (severity Med→Low)

---

## 우선순위 *(자리 — full 단계 확정)*

1. **[High] orphan ASR QA-06·QA-07** → NDP-A/B 신설 (C1)
2. **[High] 제어평면 이중 결정 명시** DP-02↔DP-04 (C2)
3. **[Med] 공유 장애도메인 FMEA·멱등 가정 승격** (C3·C4)
4. **[Med] drives↔ASR 정렬 + ★칸 KPI 재앵커** (C5·C6)

---

## 신규 결정/대안 (NDP) *(자리 → _new-dp-candidates.md)*

- **NDP-A** eval/검증 + security gate 서브시스템 DP — QA-06(red-team 하네스·injection 가드·secrets·공급망 서명) + QA-07(golden+judge 하네스) orphan을 메움. OI-7 "eval/검증 서브시스템 = 신규 DP 후보" 직접 대응.
- **NDP-B** (분리 시) Correctness 검증 하네스 DP — QA-07 전담(golden-set·judge κ≥0.8). NDP-A에서 분리할지 full에서 판정.
- **NDP-C (후보)** Agent Hierarchy Federation — DP-02 BL-2 승격(단일 Orchestrator 병목 TP-2 완화).

---

## 직전 라운드 대비 변화

- **DP 디스커션 1라운드** — 직전 review/applier 보고서 없음. prior art = dp4/review/findings.md(F-01~F-12)를 결함 유형 선례로 인용.
- findings F-01·F-02·F-03·F-04·F-08·F-10은 evaluation.md 수렴 작업으로 **부분 해소**(DP-04 본문 검증 결과 위 DP-04-*.md 판정표 참조). F-05·F-07·F-09·F-11·F-12는 **잔존** → C2~C6·각 DP Stage 2 권고로 재제기.
