# R-01 리서치 — 실행 격리 경계 4단 스펙트럼 (runc · gVisor · microVM · full VM)

> category: DP-research | for: dp3(Agent 격리 구조 — DP id 확정 전) | updated: 2026-07-31
> 목적: dp3 **축①(실행 격리 경계 등급)** 의 후보 스펙트럼을 정리한다. "container vs VM" 이지선다가 가리는 **중간 두 지대(gVisor·microVM)** 를 발굴해 결선 구도를 만드는 것이 핵심.
> 배경: dp2(Workflow 실행 구조)가 **일회용(ephemeral) 컨테이너** 모델을 확정 → dp3은 그 일회용 컨테이너의 **경계를 무엇으로 치는가**를 결정한다. 격리 등급은 축②(승인 경계 배치)의 전제("샌드박스 내부 = 자동 승인" 성립 여부)를 공급한다.
> drives: C-03(보안·안전 제약 — pass/fail 게이트), QA-03(Controllability·ASR#3), QA-01(Scalability·ASR#1 — 밀도), QA-10(Performance-E2E — 기동·I/O 세금) | constrained-by: C-01(Docker)
> 시각 자료: `docs/isolation-spectrum.html` ([Pages 배포](https://sj-meliora.github.io/archi-team-project/isolation-spectrum.html))
> ⚠️ 넘버링 주의: 이 "dp3"은 팀 재편 넘버링(dp2 = 구 DP-0004 실행구조)이며, 기존 `DP-0003-external-system-stability.md`와 **별개 결정**이다. 정식 DP id는 팀 확정 후 부여.

## 문제 정의 — 양극단의 약점

트레이드는 **"시스템콜이 공유 커널에 직접 닿는 넓은 공격면" ↔ "무거움"** 이다.

- **컨테이너(runc)**: 커널을 공유하고 namespace/cgroup **소프트웨어 규칙**으로만 구분. 컨테이너 안 프로세스가 호스트 커널의 시스템콜 **~300+개를 직접 호출** → 커널 취약점 1개면 탈출·노드 전체 함락. 벽이 규칙이지 실체가 아님.
- **full VM**: 하이퍼바이저가 하드웨어(VT-x) 수준에서 벽을 치고 게스트가 자기 커널을 통째로 부팅. 벽은 진짜인데 완전한 OS + BIOS·PCI·레거시 장치 에뮬레이션까지 끌고 와 **기동 수십 초, 메모리 수 GB** → dp2의 일회용(호출마다 생성·폐기) 모델과 경제성 불일치.

중간 두 후보는 이 트레이드를 **반대 방향에서** 공략한다.

## 조사한 격리 등급 (스펙트럼 순)

### 1) hardened container — runc (기준선 · 약)
- **벽의 재질**: 커널 SW 규칙(namespace·cgroup·seccomp).
- **비용**: 기동 ~수십 ms, 오버헤드 최소 — 밀도 최상.
- **주의점**: ⚠️ 공격면 최대. 탈출 조건 = 커널 취약점 1개(매년 발생). LLM 조종(injection) 하의 임의 코드 실행을 위협 모델에 넣으면 단독으로는 부족.
- **실사용**: Docker·K8s 기본값.

### 2) gVisor — 유저스페이스 커널 (결선 후보)
- **정의**: 앱과 진짜 커널 사이에 **가짜 커널(Sentry — 유저스페이스에서 도는 커널 재구현)** 을 끼워 시스템콜을 전부 가로챈다. 앱의 ~300+개 호출을 가짜 커널이 자기 선에서 처리하고, 진짜 커널에는 **엄선된 ~50개만** 전달(좁은 문).
- **핵심 효과**: 공격자가 컨테이너를 장악해도 손에 쥐는 건 가짜 커널 — 커널은 공유하되 **직접 접근을 차단**. 탈출하려면 가짜 커널 돌파 + 좁은 문 너머의 커널 취약점이 동시에 필요.
- **비용**: ⚠️ 시스템콜마다 가로채기 오버헤드 → **I/O 무거운 워크로드에서 세금**. 커널 재구현이라 일부 시스템콜 미지원(호환성 구멍). 기동 ~수백 ms.
- **장점**: **하드웨어 가상화 불필요** — 아무 환경에서나 구동.
- **실사용**: Google Cloud Run, GKE Sandbox(멀티테넌트 코드 실행).

### 3) microVM — Firecracker / Kata Containers (결선 후보)
- **정의**: 하드웨어 가상화 벽(VT-x)은 유지하되, VM을 무겁게 만들던 것(BIOS·PCI·레거시 에뮬레이션·범용 OS)을 제거하고 **최소 게스트 커널 + virtio 장치만** 남긴 초경량 VM.
- **핵심 수치**: Firecracker 기동 **~125 ms**, VM당 오버헤드 **~5 MB** — "VM의 벽을 컨테이너 값에".
- **Kata Containers**: 이 microVM을 **컨테이너 껍데기로 포장** — K8s `RuntimeClass` 한 줄로 해당 파드만 전용 microVM 안에서 구동. Docker/K8s UX 유지 → **C-01(Docker) 정합, Non-Risk**.
- **주의점**: ⚠️ 호스트가 가상화를 지원해야 함 — **bare-metal 또는 nested virtualization** 환경 필요(안 되는 클라우드 환경 존재). 기동·메모리는 runc보다 무거움.
- **실사용**: AWS Lambda/Fargate(Firecracker), 상용 에이전트 샌드박스 다수.

### 4) full VM (기준선 · 과잉)
- 벽의 재질은 microVM과 동일(하이퍼바이저)인데 짐(완전한 게스트 OS + 장치 에뮬레이션)이 커서 기동 수십 초·밀도 최저. **microVM에 지배**된다(같은 격리 등급을 더 싸게 얻으므로). 전통 서버 가상화 용도.

## 한 장 비교

| | runc 컨테이너 | **gVisor** | **microVM (Kata·FC)** | full VM |
|---|---|---|---|---|
| 벽의 재질 | 커널 SW 규칙 | 유저스페이스 가짜 커널(문 좁힘) | **하드웨어 가상화** | 하드웨어 가상화 |
| 커널 공유 | 직접 공유 | 공유하되 좁은 문으로만 | 게스트 커널 별도(최소) | 게스트 커널 별도(완전) |
| 탈출하려면 | 커널 취약점 1개 | 가짜 커널 + 좁은 문의 커널 취약점 | 하이퍼바이저 취약점(희귀) | 하이퍼바이저 취약점 |
| 기동 | ~수십 ms | ~수백 ms | **~125 ms – 1 s** | 수십 초 |
| 약점 | 공격면 최대 | **I/O 세금**·호환성 구멍 | 가상화 지원 필요 | 무겁고 밀도 최저 |
| 실사용 | Docker·K8s 기본값 | Cloud Run·GKE Sandbox | AWS Lambda·Fargate·Kata | 전통 가상화 |
| 역할 | **기준선(약)** | **결선 후보** | **결선 후보** | **기준선(과잉)** |

> 기동·오버헤드 수치는 대표 공개값 기반 **예시값**이며 합격선은 실측으로 확정(QA-10 budget).

## dp3에의 함의 — 결선 구도와 가르는 질문

**dp4 무브 재사용**: 양끝(runc·full VM)은 각각 "격리 부족"과 "동일 벽을 더 비싸게"로 **기준선 격하**, 최종 대결은 **gVisor(소프트웨어로 공격면 축소) vs microVM(하드웨어 벽을 컨테이너 경제성으로)**.

- **격리 등급 요구는 위협 모델에서 도출**: C-03의 injection 차단율 임계 `≥70%`는 곧 **최대 ~30%의 악성 지시가 가드레일을 통과함을 공식 인정**한 것 — 그 잔여 위협(조종당한 에이전트의 임의 코드 실행)을 봉쇄하는 층이 샌드박스다. 위협 모델에 이것이 포함되면 커널 공유 경계(runc)로는 부족하다는 논리가 성립.
- **축②와의 관계 (도출 아님·전제 공급)**: 격리 등급이 위협 모델을 담아내면 "샌드박스 내부 = 자동 승인" 원리가 성립하고, 축②는 경계 통과 지점(배포·push·credential·외부 API)별 승인 메커니즘 배정 문제로 축소된다. 격리가 불충분하면 내부에도 게이트가 필요해져 자동존이 좁아진다(TP 후보).
- **dp2 정합 (갱신 2026-07-31 — A5 확정 반영)**: dp2가 **A5(외부 오브젝트 스토리지 전달)** 를 채택(팀 입력). A5의 cold-start 지배 항은 대형 컴파일러 이미지라 샌드박스 기동(~125ms/~수백ms)은 **비변별**로 강등. 지배 변별은 **스토리지 왕복 I/O의 격리 계층 경유 세금**(아래 Q2′)으로 이동. A8 고유였던 footprint(SP-4) 되먹임은 소거.
- ⚠️ **범주 오류 주의(F-08 재발 방지)**: **microVM ≠ microkernel**. Microkernel/Plug-in(dp4 A7)은 변종 수용(Extensibility) 축의 아키텍처 패턴이고, 이 축의 어휘는 Bulkhead / Sandbox / Defense-in-depth 계열이다.

**결선(gVisor vs microVM)을 가르는 질문 — 실측 의존, 구조는 고정·수치는 측정으로:**

1. **[Q1 인프라 게이트 — 충족 확인 ✓ (2026-07-31 팀 입력)]** bare-metal 선택 가능 → microVM 계열 잔류. Q1은 변별력을 잃고 결정은 Q2′로 내려간다.
2. **[Q2′ 성능 게이트 — dp2=A5 확정 반영 재정의]** dp2가 A5(claim-check)를 채택해 20GB는 **단계마다 스토리지 read → 로컬 스크래치 → write 왕복**으로 흐른다(구 Q2의 "로컬 볼륨" 전제 폐기). 격리 계층은 이 경로에 곱해지는 세금 — gVisor는 네트워크(netstack)·파일(gofer) syscall 이중 가로채기, microVM은 virtio near-native. **격리 계층 경유 후에도 전달세금이 QA-10 ≤5% budget에 남는가?** 못 남으면 microVM 우위.
3. **[부차 변별 — Q2′ 동률 시]** gVisor syscall 호환성 구멍(컴파일러·양자화 toolchain smoke test 필요) ↔ microVM 밀도·운영 성숙도.

## 출처
- [gVisor — Architecture Guide](https://gvisor.dev/docs/) — Sentry(유저스페이스 커널)·시스템콜 가로채기 구조
- [Firecracker microVM](https://firecracker-microvm.github.io/) — ~125 ms 기동·~5 MB 오버헤드, AWS Lambda/Fargate 기반
- [Kata Containers](https://katacontainers.io/) — microVM + OCI/K8s RuntimeClass 컨테이너 UX
- [Kubernetes — Runtime Class](https://kubernetes.io/docs/concepts/containers/runtime-class/) — 파드 단위 런타임(격리 등급) 선택 메커니즘
