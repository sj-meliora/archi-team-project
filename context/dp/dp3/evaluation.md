# dp3 통합 평가 — 축①·축② 수렴 결론 (microVM + 3단 승인 배정)

> category: DP-eval | for: dp3(Agent 격리 구조 — DP id 확정 전) | updated: 2026-07-31
> 전제(팀 입력, INDEX Iter2): **dp2 = A5 계열(외부 오브젝트 스토리지 전달) 채택** · **bare-metal 선택 가능**.
> 평가 기준: ASR(QA-01 Scalability, QA-02 Availability, QA-03 Controllability, QA-07 Correctness) + QA-10(Performance-E2E) + C-03(제약 — pass/fail, ★ 비교 미적용) + C-01(Docker).
> ⚠️ 별점은 **"격리 계층이 해당 QA에 끼치는 영향" 기준의 예시 캘리브레이션**이며 Q2′ 마이크로벤치로 확정한다.

## 축① — gVisor vs microVM 기준별 판정

### 기준별 논거

| 기준 | gVisor | microVM (Kata/FC) | 판정 |
|---|---|---|---|
| **Performance-E2E** (QA-10) | 20GB×2/단계가 네트워크(netstack)·파일(gofer) **이중 가로채기** 통과 — A5의 유일 약점(전달세금 ≤5% budget)을 정확히 증폭 | virtio-net/blk **near-native** | **microVM 결정적 우위** |
| **Availability** (QA-02) | 산출물 내구성은 dp2(A5 스토리지)가 담당 → 격리 등급 비변별. 단 **syscall 호환성 구멍 = 단계 실행 중 실패 리스크** | 자기 커널이라 호환성 완전, Lambda 규모 실증 | microVM 약우위 |
| **Controllability** (QA-03) | 중단 ack/graceful stop·권한 게이트는 제어평면(DP-01/0003) 소관 → **비변별**. C-03 봉쇄 논증(injection 잔여 ~30%)은 공유 커널이라 논증 부담↑ | 하드웨어 벽이라 봉쇄 논증 단순("하이퍼바이저 취약점 필요") | microVM 약우위 (본질은 C-03 assurance) |
| **Scalability** (QA-01) | netstack 패킷 처리 CPU 세금이 A5의 knee(공유 스토리지 대역폭)를 실효적으로 앞당김 | 오버헤드 ~5MB+최소 게스트 커널 — 밀도 영향 미미 | microVM 약우위 |
| **Correctness** (QA-07) | 컴파일러·양자화 toolchain의 다양한 syscall 사용 → 미지원 구멍 가능 | 완전한 커널 → toolchain 무수정 구동 | microVM 우위 |
| C-01 / 인프라 | 어디서나 구동(장점이었음) | Kata RuntimeClass로 C-01 충족 + **bare-metal 가용으로 Q1 해소** | **동률 (gVisor 유일 강점 소거)** |

### ★ 매트릭스 (격리 계층의 영향 기준)

| 대안 | Perf-E2E (QA-10) | Avail (QA-02) | Controll (QA-03) | Scal (QA-01) | Correct (QA-07) | 비고 |
|---|:---:|:---:|:---:|:---:|:---:|---|
| runc (기준선·약) | ★★★ | ★★★ | ★★★ | ★★★ | ★★★ | ⚠️ C-03 위협 모델(injection 잔여의 임의 코드 실행) 봉쇄 논증 곤란 → **제약 게이트에서 격추** — ★ 무의미 |
| **gVisor** | ★☆☆ | ★★☆ | ★★★ | ★★☆ | ★★☆ | A5 경로 이중 세금·호환성 구멍 |
| **microVM (채택 권고)** | **★★★** | **★★★** | **★★★** | **★★★** | **★★★** | Q2′ 벤치 = budget 통과 확인용 |
| full VM (기준선·과잉) | ★★★ | ★★★ | ★★★ | ★☆☆ | ★★★ | 기동 수십 초·밀도 최저 — **microVM에 지배** |

### 핵심 논리 — gVisor의 승리 시나리오가 입력으로 소거됨
gVisor가 이기는 시나리오는 원래 둘뿐이었다: ① **가상화 불가 환경** → bare-metal 가용 입력으로 소거. ② **I/O 가벼운 워크로드 + 밀도 절대 우선** → dp2=A5 확정으로 워크로드가 정반대(단계마다 20GB 왕복이 지배 경로)가 되어 소거. 남은 전 기준에서 microVM이 동률 또는 우위 → **방향은 microVM으로 확정, Q2′ 마이크로벤치는 "선택 근거"가 아니라 "budget 통과 확인용"으로 강등**.

## 축② — 승인 배정 기준별 판정

| 기준 | 전면 HITL (기준선) | **3단 티어 배정 (권고)** | 전면 자동 (기준선) |
|---|---|---|---|
| Controllability (QA-03) | 명목상 최대 — 그러나 일 ~50건 승인 → **rubber-stamping으로 게이트 실질 무력화** | C-03 게이트(고위험 HITL 100%) 충족 + 승인이 드물어 실질 검토 유지 | **C-03 위반 — 부적격** |
| Availability (QA-02) | 사람 부재 = 파이프라인 블로킹 | 일상 경로는 standing 정책 — 사람 대기 없음 | — |
| Scalability (QA-01) | 사람 = USL α항, workload에 비례 | 에스컬레이션 ∝ 산출물 수(production 승격만) — **사람 병목 분리** | — |
| Performance (QA-10) | 승인 대기가 크리티컬 패스 진입 | 승인이 off-path(production 승격 시점만) | — |

### 권고 배정표 (초안 — 인터페이스 열거 완전성은 미결)
| 경계 통과 인터페이스 | 승인 메커니즘 |
|---|---|
| 샌드박스 내부 (노드 실행·중간 처리) | **자동** (가역 — 롤백 = 컨테이너 폐기) |
| 공유 오브젝트 스토리지 write | **정책 사전승인** — 워크플로별 스코프 자격증명(버킷/프리픽스 최소권한) + C-03 SLSA 서명 |
| 내부(canary) 배포 · 허용된 외부 API | **정책 사전승인** |
| production 배포 승격 · credential 사용 · delete | **액션별 HITL** (C-03 게이트 100%) |

## 조합의 ATAM적 의미 — TP-1의 최적 사용
채택 조합(**microVM + 3단 배정**)은 TP-1(격리 강도 ↔ 자동존 폭)을 최적으로 쓴다: **가장 강한 벽(하드웨어)이 가장 넓은 자동존을 떠받치고**, 그 자동존이 ASR 1순위(QA-01)가 요구하는 "사람 개입 없는" 처리량을 만든다. 반대 조합은 각각 격추된다 — (약한 벽 + 넓은 자동존) → C-03 봉쇄 논증 실패, (강한 벽 + 전면 HITL) → QA-01 사람-α항 병목 + rubber-stamping.

## 수렴 결론
- **축① = microVM (Kata Containers / Firecracker)** — runc·full VM은 기준선(각각 제약 게이트 격추·지배당함), gVisor는 전 기준 열세로 탈락.
- **축② = 3단 위험 티어 배정** — 위 배정표. 전면 HITL·전면 자동은 기준선(각각 QA-01/rubber-stamping·C-03 부적격).

## 뒤집힘 조건 (기록)
1. **bare-metal 전제 철회** → Q1 부활 → gVisor로 회귀.
2. **Q2′ 벤치에서 virtio 경유조차 5% budget 미달** → 격리 등급이 아니라 **dp2 전달 구조의 문제로 격상**(dp2 재론 — 격리 계층 교체로 해결 불가).
3. 이 두 조건 외에는 microVM 선택이 안정적.

## 미해결 / 검증 필요 (INDEX 미결과 연동)
- **Q2′ 마이크로벤치 (확인용)**: 20GB read/write 왕복을 runc(기준)·gVisor·Kata 계층별 통과 — QA-10 5% budget 대조. dp2 PoC(mock 파이프라인)와 통합 가능.
- **배정표 인터페이스 열거 완전성**: 본 시스템 기준 경계 통과 지점 전수 열거 — C-03 "우회 0건" 논증 상한, OI-7 red-team 하네스 검증 대상.
- **DP-01·DP-0003 계층 관계 명시** (게이트 존재=기존 DP, dp3=배치·입도) — 이중 소유권 방지.
- **DP id 확정** (기존 DP-0003과 충돌 회피) 후 본문(DP 문서) 작성 — 본 evaluation의 수렴 결론·별점·뒤집힘 조건을 본문 ATAM 절로 승격.
