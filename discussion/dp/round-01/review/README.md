# discussion/dp/round-01/review — 내비게이션

> red team(Reviewer) 산출물. DP 디스커션 **round-01** · scope 현재: **sample**(골격 + 대표 2개).
> 방법론: `discussion/dp/Reviewer.md` · 공통 프로토콜: `discussion/README.md`

## 파일

| 파일 | 역할 | 상태 |
|---|---|---|
| `report.md` | 라운드 종합 결론 — Traceability Matrix(DP×ASR)·orphan ASR·판정표·교차발견(C*)·우선순위·NDP | **골격** (Matrix 완성, 판정표는 샘플 2개) |
| `DP-04-workflow-execution-structure.md` | DP-04 3렌즈 상세 (대표 샘플 1) | ✅ 완료 |
| `DP-02-agent-hierarchy.md` | DP-02 3렌즈 상세 (대표 샘플 2) | ✅ 완료 |
| `_new-dp-candidates.md` | 신규 결정/대안(NDP-*) — orphan ASR을 메울 후보 | 자리(placeholder) |
| `DP-01-*.md` / `DP-03-*.md` / `DP-05-*.md` | 나머지 DP 상세 | **full 단계** |

## 읽기 순서
1. `report.md` — Traceability Matrix·orphan ASR(QA-06/07)·판정표 한눈에
2. `DP-04-*.md` — "ASR 1개만 cover, 강점이 비-ASR에 쏠림"의 대표 사례
3. `DP-02-*.md` — "핵심 ASR 다수 cover + Standby 3안·KPI 앵커" 대표 사례
4. `_new-dp-candidates.md` — orphan을 메울 NDP

## sample 단계에서 고른 항목
- **DP-04**(가장 풍부 — dp4 A5 vs A8 수렴 + findings F-01~F-12 prior art) · **DP-02**(핵심 ASR QA-01/02/03 다수 cover + Standby 3안).
- 나머지(DP-01·03·05) 상세 + 교차발견(C*) 본문화 + NDP 확정은 **full 단계**로 이월.

## ASR (= QA-01~07, DP 생성 동인)
QA-01 Scalability · QA-02 Availability · QA-03 Controllability · QA-04 Observability · QA-05 Efficiency · QA-06 Security/Safety · QA-07 Correctness
> orphan(미cover) = **QA-06 · QA-07** (둘 다 H 중요도).
