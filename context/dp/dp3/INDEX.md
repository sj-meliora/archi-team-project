# dp3 Agent 격리 구조 — 설계 작업 색인

> category: DP-meta | owner: 17조(담당: 본인) | updated: 2026-07-31
> 목적: dp3(Agent 격리 구조) 설계 작업의 색인·iteration 로그·상태표.
> ⚠️ **넘버링 주의**: 이 "dp3"은 팀 재편 넘버링(dp2 = 구 DP-0004 실행구조)이며, 기존 `DP-0003-external-system-stability.md`와 **별개 결정**. 정식 DP id는 팀 확정 후 부여.

## 결정 포인트 (재정식화 후)
각 Agent의 실행 격리 구조를 어떻게 설계하는가 — **두 서브결정**:

- **축① 실행 격리 경계 등급**: 일회용 컨테이너(dp2 확정)의 경계를 무엇으로 치는가. `runc ↔ gVisor ↔ microVM ↔ full VM` 스펙트럼에서 양끝은 기준선 격하, **결선 = gVisor vs microVM**.
- **축② 승인 경계 배치**: 경계 통과 지점(배포·push·credential·외부 API)마다 `자동 | 정책 사전승인 | 액션별 HITL`을 배정. 전면 HITL·전면 자동은 기준선 격하(후자는 C-03 위반 부적격).

> 원래 팀 프레임 "(1) container vs VM (2) HITL 강제 vs 자동 승인"을 재정식화한 경위·근거는 `decision-axes.md` §왜 재정식화가 필요했나.

- **driving**: C-03(제약 게이트) · QA-03(ASR#3) · QA-01(ASR#1) · QA-10 | constrained-by: C-01(Docker)
- **인접 정합**: dp2(cold-start·footprint 되먹임) · DP-01/DP-0003(HITL·권한 게이트 소유권 — dp3은 배치·입도만) · OI-7(red-team 하네스)

## 폴더 구조
```
dp3/
├── INDEX.md            (이 파일) 색인·iteration 로그
├── decision-axes.md    두 서브축 정의·축 간 관계(전제 공급·TP-1)·본문 작성 가이드
├── R-01-isolation-boundary-spectrum.md    축① 리서치 — 격리 4단 스펙트럼
└── R-02-approval-boundary-mechanisms.md   축② 리서치 — 승인 3단 메커니즘·CC 레퍼런스
```

## 산출물 상태표
| 문서 | 내용 | 상태 |
|---|---|---|
| R-01 | 축① 스펙트럼(runc·gVisor·microVM·full VM), 결선 가르는 Q1/Q2 | 완료 |
| R-02 | 축② 3단 메커니즘, 승인 경계=격리 경계 원리, CC 로컬/클라우드 레퍼런스 | 완료 |
| decision-axes.md | 두 축 정의, 축 간 관계(도출 아님·전제 공급), 소유권 정합, ATAM 요소 후보 | 완료 |
| 본문 (dp3 DP 문서) | 결선 비교·별점·ATAM(SP/TP/Risk/NR)·결정 질문 통합 | **예정** (DP id 확정 후) |
| 시각 자료 | `docs/isolation-spectrum.html` — 축① 4단 스펙트럼 다이어그램 ([Pages](https://sj-meliora.github.io/archi-team-project/isolation-spectrum.html)) | 완료 |

## Iteration 로그
- **Iter1 (2026-07-31)**: 원 프레임의 문제 3개 식별 — ① 축(2) 양끝이 SSoT에 격추됨(전면 자동=C-03 위반 부적격 / 전면 HITL=시스템 전제·QA-01 위배) ② 축(1) 이지선다가 지배적 중간(gVisor·microVM) 은폐 ③ 두 축의 범주 혼동(비인가 능력 vs 인가된 오판단). → 재정식화: 독립 서브결정 2개 + TP-1 연결. R-01·R-02·decision-axes 작성, 축① 다이어그램 시각화·Pages 배포. 레퍼런스: Claude Code 로컬/클라우드 승인 비대칭(R-02).

## 미결 (본문 작성 시 처리)
- **DP id 부여** (기존 DP-0003과 충돌 회피 — 팀 확정).
- **경계 통과 인터페이스 열거** (본 시스템 기준 — push/배포/credential/외부 API/아티팩트 + α). 열거 완전성 = C-03 우회0 논증 상한 → OI-7 red-team 하네스 검증 대상.
- **축② 배정표 초안** (canary=정책 사전승인 / production 승격=HITL 등) + 발생률·승인 SLA 실측 항목 정의.
- **DP-01·DP-0003과의 계층 관계 명시** (게이트 존재=기존 DP, dp3=배치·입도) — 이중 소유권 방지, open-issues 등재.
- 축① Q1(인프라)·Q2(gVisor 20GB I/O 세금) 실측 계획 — dp2 PoC(mock 파이프라인)와 통합 가능성.
