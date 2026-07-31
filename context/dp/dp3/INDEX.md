# dp3 Agent 격리 구조 — 설계 작업 색인

> category: DP-meta | owner: 17조(담당: 본인) | updated: 2026-07-31
> 목적: dp3(Agent 격리 구조) 설계 작업의 색인·iteration 로그·상태표.
> ⚠️ **넘버링 주의**: 이 "dp3"은 팀 재편 넘버링(dp2 = 구 DP-0004 실행구조)이며, 기존 `DP-0003-external-system-stability.md`와 **별개 결정**. 정식 DP id는 팀 확정 후 부여.

## 결정 포인트 (재정식화 후)
각 Agent의 실행 격리 구조를 어떻게 설계하는가 — **두 서브결정**:

- **축① 실행 격리 경계 등급**: 일회용 컨테이너(dp2 확정)의 경계를 무엇으로 치는가. `runc ↔ gVisor ↔ microVM ↔ full VM` 스펙트럼에서 양끝은 기준선 격하, **결선 = gVisor vs microVM**. Q1(인프라)은 충족 확인으로 해소 — **결정은 Q2′(격리 계층 경유 전달세금)**.
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
| evaluation.md | 기준별 판정·★ 매트릭스·수렴 결론(**microVM + 3단 배정**)·뒤집힘 조건 | 완료 |
| 본문 (dp3 DP 문서) | evaluation 수렴 결론을 ATAM 절로 승격한 정식 DP 문서 | **예정** (DP id 확정 후) |
| 시각 자료 | `docs/isolation-spectrum.html` — 축① 4단 스펙트럼 다이어그램 ([Pages](https://sj-meliora.github.io/archi-team-project/isolation-spectrum.html)) | 완료 |

## Iteration 로그
- **Iter1 (2026-07-31)**: 원 프레임의 문제 3개 식별 — ① 축(2) 양끝이 SSoT에 격추됨(전면 자동=C-03 위반 부적격 / 전면 HITL=시스템 전제·QA-01 위배) ② 축(1) 이지선다가 지배적 중간(gVisor·microVM) 은폐 ③ 두 축의 범주 혼동(비인가 능력 vs 인가된 오판단). → 재정식화: 독립 서브결정 2개 + TP-1 연결. R-01·R-02·decision-axes 작성, 축① 다이어그램 시각화·Pages 배포. 레퍼런스: Claude Code 로컬/클라우드 승인 비대칭(R-02).
- **Iter2 (2026-07-31)**: **dp2 결정 입력 반영** — dp2 = A5 계열(외부 오브젝트 스토리지 전달) 채택 + bare-metal 가용(팀 입력). → 축① **Q1 해소**(microVM 잔류·변별력 상실), **Q2′ 재정의**(로컬 볼륨 전제 폐기 → 스토리지 왕복의 격리 계층 경유 세금: gVisor netstack+gofer 이중 vs microVM virtio), 부차 변별(toolchain 호환성 ↔ 밀도·성숙도) 추가. 축②: 스토리지 write를 경계 통과 인터페이스 1급 등재 — **정책 사전승인 티어**(스코프 자격증명 + C-03 서명), durable state 외부화로 "내부=자동 승인" 전제 강화(QA-02 정합).
- **Iter3 (2026-07-31)**: **통합 평가·수렴**(`evaluation.md`) — ASR 기준별 판정 결과 gVisor의 승리 시나리오 2개(가상화 불가 환경·I/O 가벼운 워크로드)가 모두 팀 입력으로 소거됨을 확인 → **축① microVM(Kata/FC) 채택 권고**(Q2′ 벤치는 확인용 강등), **축② 3단 티어 배정 권고**(배정표 초안 포함). 뒤집힘 조건 2개(bare-metal 철회 → gVisor / virtio도 5% 미달 → dp2 재론) 기록.

## 미결 (본문 작성 시 처리)
- **DP id 부여** (기존 DP-0003과 충돌 회피 — 팀 확정).
- **경계 통과 인터페이스 열거** (본 시스템 기준 — push/배포/credential/외부 API/아티팩트 + α). 열거 완전성 = C-03 우회0 논증 상한 → OI-7 red-team 하네스 검증 대상.
- **축② 배정표 초안** (canary=정책 사전승인 / production 승격=HITL 등) + 발생률·승인 SLA 실측 항목 정의.
- **DP-01·DP-0003과의 계층 관계 명시** (게이트 존재=기존 DP, dp3=배치·입도) — 이중 소유권 방지, open-issues 등재.
- **축① Q2′ 실측 계획** — 스토리지 왕복(20GB read/write)을 격리 계층별(runc 기준·gVisor·Kata)로 통과시키는 마이크로벤치 + 컴파일러/양자화 toolchain smoke test(gVisor 호환성). dp2 PoC(mock 파이프라인)와 통합 가능성.
- **DP-0004(dp2) 본문 반영** — A5 계열 채택 확정을 DP-0004 본문 status(현 "결정대기")에 기록하는 것은 dp3 범위 밖 별도 작업(SSoT 변경 — 팀 승인 후).
