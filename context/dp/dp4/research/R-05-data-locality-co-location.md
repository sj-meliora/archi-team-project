# R-05 리서치 — Data-Locality Scheduling + Co-location / Pipeline Parallelism

> category: DP-research | for: DP-0004 | updated: 2026-06-22
> 목적: 2안(로컬 전달·격리) 계열의 **약점(확장 단위=Workflow 통째, R-2)을 고치면서 로컬 전달을 유지**하는 실행 기반을 조사. "데이터를 컴퓨트로 옮기는" 방향(대용량 산출물 20GB+에 유리).
> 배경: `decision-axes.md`의 2x2에서 "일회용 × 로컬" 칸이 비어 있어, 그 칸(A8)을 떠받칠 패턴군을 발굴.

## 조사한 패턴

### 1) Data-Locality Scheduling (데이터 지역성 우선 스케줄링)
- **정의**: 큰 데이터를 컴퓨트로 옮기는 대신, **컴퓨트(작업)를 데이터가 있는 노드로 보낸다**. 입력이 큰 분산 처리(HDFS/Spark/MapReduce)의 기본 원칙.
- **핵심 효과**: 단계 간 대용량 전달이 네트워크/오브젝트 스토리지를 타지 않고 **노드 로컬 디스크**에서 끝남 → 전달 오버헤드 최소화. 20GB+ 산출물이 4단계를 지나는 본 과제에 직접 유효.
- **메커니즘**: 스케줄러가 작업을 **데이터 affinity**로 배치(가능하면 로컬, 불가하면 rack-local, 최후에 원격). = **locality-first, 넘치면 spill**.
- **주의점**: ⚠️ 로컬 노드 용량 초과 시 **로컬리티가 깨지고** 원격 전달로 degrade. 로컬 디스크는 **내구성 약함**(노드 사망 시 유실).

### 2) Co-location / Affinity / Gang Scheduling
- **정의**: 함께 동작해야 할 작업들을 **같은 노드(또는 노드 그룹)에 배치**. Kubernetes의 pod affinity / volume locality / (필요시) gang scheduling으로 한 모델의 파이프라인 단계들을 한 노드에 묶음.
- **효과**: 한 모델의 4단계가 같은 로컬 볼륨(emptyDir/로컬 PV)을 공유 → 단계 간 전달이 로컬. 격리는 **단계 컨테이너 단위 Bulkhead**로 유지.

### 3) Pipeline Parallelism + Bounded Buffer (단계 병렬 + 로컬 backpressure)
- **정의**: 파이프라인을 **단계별 worker 풀**로 운영하되, 단계 속도 차이에 맞춰 **풀 개수를 비대칭**으로 둔다(병목 단계만 worker↑). 단계 간은 **bounded buffer**(유한 큐)로 연결해 로컬 backpressure 확보.
- **효과**: 2안처럼 파이프라인을 통째 복제하지 않고, **노드 안에서 병목 단계 worker 수만** 늘려 유휴(노는 Converter) 제거 → R-2(활용률<70%) 해소. (Unix pipe의 in-memory 스트리밍, 스테이지-병렬 처리와 동계열)

## DP-0004에의 함의 — 비어 있던 "일회용 × 로컬" 칸을 채움
- 1안/A5(원격 계열)는 전달세금(QA-10)을 내고 확장·격리를 얻음.
- 본 패턴군은 **로컬 전달을 지키면서**(전달세금 회피) **단계 입도 확장**(R-2 해소)을 가능케 함 → 2안의 진화형.
- 단, **노드 용량 한계**로 절대적 로컬이 아니라 locality-first(넘치면 spill) → 규모가 커지면 A5에 수렴(`decision-axes.md` 스펙트럼 통찰).

→ 도출 대안: **[A8] Co-located Ephemeral Stage (locality-first)**.

## 출처
- [Data Locality in Hadoop/Spark (개념)](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html) — "moving computation is cheaper than moving data"
- [Kubernetes — Assigning Pods to Nodes (affinity/anti-affinity)](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
- [Kubernetes — Volume node affinity / local PersistentVolume](https://kubernetes.io/docs/concepts/storage/volumes/#local)
