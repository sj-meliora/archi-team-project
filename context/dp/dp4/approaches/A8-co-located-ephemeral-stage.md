# A8 (8안) Co-located Ephemeral Stage — 로컬 핀 일회용 단계 실행

> category: DP-approach | for: DP-0004 | status: 발굴·평가완료 | updated: 2026-06-22
> 근거 리서치: R-05 | drives: QA-01(주), QA-10, QA-08, QA-12
> 분류: `decision-axes.md` 2x2의 "일회용 × 로컬" 칸 (그동안 비어 있던 칸). 2안의 진화형.

## 구조
한 모델의 파이프라인 4단계(IR Converter→Graph Optimizer→Quantizer→Compiler)를 **데이터가 있는 노드에 핀(data-affinity)으로 박아** 실행한다. 단계 작업은 호출마다 **일회용 컨테이너**로 띄우되(=A5의 수명 모델), 스케줄러가 **이전 단계가 20GB를 남긴 그 노드** 위에 배치한다(=2안의 로컬 전달). 단계 간 전달은 오브젝트 스토리지가 아니라 **노드 로컬 볼륨/스트리밍**. 노드 안에서는 단계별 worker 풀의 **개수를 비대칭**으로(병목 단계만 ↑) 두고 bounded buffer로 연결한다. 로컬 노드 용량 초과 시에만 원격으로 spill 한다(**locality-first**).

## 근거 tactic/pattern
- **Data-Locality Scheduling**("컴퓨트를 데이터로"), **Co-location / Pod Affinity / Volume Locality**, **Pipeline Parallelism(단계 비대칭 worker)**, **Bounded Buffer(로컬 backpressure)**, **Ephemeral instance**, **Bulkhead(단계 컨테이너 격리)**.

## 기존 안과의 차별점
- **2안 대비**: 확장 단위를 "Workflow 통째"에서 **"단계 worker 개수"** 로 잘게 쪼개 R-2(노는 Converter 복제) 해소. 로컬 전달·격리는 유지.
- **A5 대비**: 수명 모델(일회용)은 동일하나, 배치를 **클러스터 자유 → 데이터 있는 노드 핀**으로 제약. → 전달세금(claim-check 20GB 왕복) 회피. A5↔A8은 규모에 따라 수렴하는 **양 끝**(`decision-axes.md`).

## QA별 장점 / 단점
- **[Scalability QA-01] ★★★ (주 강점)**: 병목 단계 worker만 노드 내에서 탄력 확장(비대칭 풀) → 유휴 제거, 활용률↑(≥70% 직접 겨냥). 일회용이라 scale-to-zero도 가능.
- **[Performance-E2E QA-10] ★★★**: 단계 간 전달이 로컬 디스크/스트리밍 → **claim-check 20GB 왕복 회피**(2안의 유일 강점 계승). ⚠️ 단 일회용 cold-start는 잔존(pre-warm 완화), 노드 초과 spill 시 원격 전달로 degrade.
- **[Reliability-Workflow QA-08] ★★☆**: 단계 컨테이너 단위 Bulkhead로 장애 국소화. ⚠️ 단 **로컬 디스크는 내구성 약함** — 노드 사망 시 그 위 산출물 유실 → 해당 단계부터 재실행(A5의 오브젝트 스토리지 보존 대비 복구 불리).
- **[Maintainability QA-12] ★★☆**: 단계 컨테이너 독립 교체. ⚠️ 단 data-affinity 스케줄링·로컬 볼륨 수명관리 등 배치 제어 복잡성 추가.

## Trade-off 별점
| Performance | Scalability | Reliability-WF | Maintainability |
|:---:|:---:|:---:|:---:|
| ★★★ | ★★★ | ★★☆ | ★★☆ |

## mini-ATAM
- **SP**: ① 노드 용량 대비 모델당 파이프라인 footprint가 QA-10(로컬 유지 vs spill)에 강하게 민감. ② 로컬 디스크 내구성 정책(복제/체크포인트 여부)이 QA-08에 민감.
- **Risk**: 모델 폭증으로 노드 초과 시 **로컬리티 붕괴 → 원격 전달 degrade**(A8 강점 소멸, A5에 수렴). 로컬 디스크 유실로 재실행 비용↑. data-affinity 스케줄러 구현 난이도.
- **Non-Risk**: C-0001(Docker)·C-0002(이식성) — K8s Job + pod affinity/local volume로 컨테이너 기반 구동 가능.

## 인접 DP 정합
- DP-0001 2안(Dynamic Agent Pool)과 정합 — Agent를 노드 핀 ephemeral invocation으로. DP-0005 캐시는 로컬 볼륨 계층과 결합(노드 로컬 memoization). A4(무상태 filter)를 **로컬 스트리밍 모드**로 얹으면 A8의 단계 분해와 자연 결합.
