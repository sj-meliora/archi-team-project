# PoC 통합 계획 — round-01 counsel

> 각 QA 권고의 PoC를 모아 우선순위·의존성·공유 자산·총 기간을 정리한다. 방법론: [`../../Council.md`](../../Council.md) §5 · 종합: [`counsel.md`](counsel.md)
> date: 2026-06-24 · 원칙: 작게 시작(앵커 50~100, 부하 수십 VU), 레퍼런스 수치 복제 금지, 각 PoC는 silent cap 명시.

## PoC 목록 (아키타입별)

| PoC | 증명 대상 | 아키타입 | 출처 QA | 합격선(요약) |
|---|---|---|---|---|
| PoC-S1 | scaling efficiency ≥ 0.8 | 부하시험 | QA-01 | 부하 2배 → 처리량 ≥1.8배 |
| PoC-S2 | rate-limit 헤드룸이 진짜 병목 | 부하시험 | QA-01 | admission control로 throttle 0 |
| PoC-P1 | speedup × first-pass 품질 게이트 | eval+측정 | QA-07 | 노드타입별 speedup, 커버리지 ◯% |
| PoC-P2 | runner 오버헤드 분리 계측 | 계측 | QA-07 | trace 완전성 ~100%, warm 절감 |
| PoC-A1 | 무손실·멱등 재개 | 장애주입(chaos) | QA-02 | 손실 0·중복 0·재개 ≤1분 |
| PoC-A2 | 외부 LLM 장애 graceful degradation | 장애주입(chaos) | QA-02 | 자동 재개율 ◯%, backpressure 작동 |
| PoC-C1 | 중단 ack/graceful 정지 분리 | 측정 | QA-03 | ack ≤5초, 정지완료 ≤◯초 |
| PoC-C2 | 위반 통과 0 + runaway cap | eval(적대적) | QA-03 | 위반 0, cap 100% |
| PoC-O1 | span/event-history 완전성 | 계측 | QA-04 | 안전 100%/일반 ≥95%, MTTD ≤◯ |
| PoC-E1 | prompt caching $/task·TTFT 절감 | 비용 A/B | QA-05 | $/task 유의 감소, 캡 ◯ 산출 |
| PoC-R1 | noisy-neighbor 쿼터 격리 | 격리 주입 | QA-06 | 타 WF 중단 ≤1%, 쿼터 침범 0 |
| PoC-E2E1 | E2E latency·throughput·전달 5% | 부하시험 | QA-08 | A5 vs A8 전달 5% 산식 판정 |
| PoC-K1 | 캐시 우회 pass^k·유효-결정률 | 반복시행 | QA-09 | pass^k 측정, 캐시 치트 대조 입증 |
| PoC-M1 | CIS p95 + 모델교체 무중단 | 측정 | QA-10 | CIS p95 ≤◯, 무중단 100% |
| PoC-N-A1 | 권한위반·injection·공급망 | eval(적대적) | NQA-A | 위반 0, 차단율 ◯%, 무결성 100% |
| PoC-N-B1 | golden set + LLM-as-judge 정답률 | eval 하네스 | NQA-B | judge 인간 일치 ≥목표, 정답률 측정 |
| PoC-N-C1 | $/완료모델 분해·절감률 | 비용 A/B+계측 | NQA-C | $/모델 분해, baseline 절감률 |

> §4 라이브러리 9개 유형(아키타입)을 모두 덮음: 부하시험·chaos·격리주입·계측·eval 하네스·반복시행·비용 A/B.

## 의존 그래프 (선행 → 후행)

```
[기반 인프라 PoC — 먼저]
PoC-O1 (OTel span + event-history)  ──┐ 측정·비용 PoC의 계측 토대
                                       ├─→ PoC-E1, PoC-N-C1 (토큰/비용 계측 사용)
                                       └─→ PoC-P2 (오버헤드 span)

[정확성 게이트 — 교차 의존의 허브]
PoC-N-B1 (golden set + judge)  ──┬─→ PoC-P1 (first-pass 성공 판정 = NQA-B 게이트)
                                  ├─→ PoC-K1 (유효-결정률 판정 = NQA-B 게이트)
                                  └─→ PoC-N-C1 (성공분만 비용 집계)

[적대적 eval 하네스 — 공유 자산]
PoC-C2 (QA-03 위반·runaway)  ≡  PoC-N-A1 (NQA-A 보안)  ── 하나의 red-team 하네스로 통합

[durable execution 위에]
PoC-A1·A2 (chaos)  ──→ event-history가 PoC-O1과 동일 엔진 공유

[독립 부하시험 — 병렬 가능]
PoC-S1·S2 (QA-01)  ‖  PoC-E2E1 (QA-08)  ‖  PoC-R1 (QA-06)  ── 같은 부하 하네스(k6/Locust) 공유
PoC-E2E1 ──→ DP-0004 A5 vs A8 택일 산식에 데이터 공급
```

**핵심 교차 의존 (명시 요구사항)**:
- **QA-07 ↔ NQA-B**: PoC-P1의 first-pass 성공 판정은 PoC-N-B1(정확성 게이트)에 의존 → **NQA-B 선행**.
- **QA-09 ↔ NQA-B**: PoC-K1의 유효-결정률 판정도 NQA-B 게이트 의존 → **NQA-B 선행**.
- **QA-03 ≡ NQA-A**: 적대적 eval 하네스 공유(PoC-C2 = PoC-N-A1 확장).
- **QA-04 → 비용 PoC**: PoC-O1의 토큰 계측(`gen_ai.usage.*`)을 PoC-E1·N-C1·N-B1·N-C1이 재사용 → **QA-04 계측 선행**.

## 우선순위 (3 그룹)

**Group 1 (토대 — 먼저 깔아야 나머지가 측정됨)**
1. PoC-O1 (관측 계측) — 토큰·span을 모든 비용·성능 PoC가 재사용.
2. PoC-N-B1 (정확성 게이트) — QA-07·QA-09 KPI를 닫는 전제.
3. PoC-A1 (durable execution chaos) — event-history가 O1과 엔진 공유, 가용성 핵심.

**Group 2 (High verdict 직결 — 발표 핵심)**
4. PoC-S1·S2 (QA-01 측정불가 KPI 교체).
5. PoC-P1·P2 (QA-07 측정불가 KPI + 품질 게이트, N-B1 후행).
6. PoC-E2E1 (QA-08 정의↔KPI 정렬 + DP-0004 택일 데이터).
7. PoC-C2/N-A1 (제어·보안 적대적 eval — 채택 강력권장 NQA-A).

**Group 3 (보강·Med/Low)**
8. PoC-A2(외부 장애), PoC-R1(격리), PoC-E1(캐싱), PoC-N-C1(비용), PoC-K1(일관성), PoC-C1(중단 latency), PoC-M1(유지보수).

## 공유 자산 (한 번 만들어 여러 PoC가 사용)
- **부하 하네스(k6/Locust + mock 4단계 파이프라인)**: S1·S2·E2E1·R1.
- **durable 엔진(event-history)**: A1·A2·O1·K1(버전 핀닝).
- **적대적 eval 세트**: C2·N-A1 (OWASP LLM Top-10 매핑 권장).
- **golden set + LLM-as-judge**: N-B1 → P1·K1·N-C1.
- **OTel GenAI 계측**: O1 → E1·N-C1·P2.

## 총 기간 (러프 추정)
- Group 1: 약 1.5~2주(계측·golden set 구축이 최장 — golden set 4단계 라벨이 critical path).
- Group 2: 약 2주(부하 하네스·eval 공유로 병렬화).
- Group 3: 약 1.5주.
- **합계 약 5~6주** (공유 자산 재사용으로 단축 가정). PoC는 "전부 검증" 착시 방지 — 각 PoC의 silent cap을 개별 파일에 log함.

## PoC가 못 보는 것 (전역 silent cap 집계)
- **mock 파이프라인의 현실성**: compute 시간 분포·20GB 대역폭·노드 사양은 실측 의존(DP-0004 산식은 PoC로 구조만 고정, 수치는 실환경 필요).
- **외부 시스템 통합**: Jira·빌드서버·vault 멱등성·자격증명은 통합 환경에서만 검증.
- **eval/golden/red-team 커버리지 = 신뢰 상한**: 미포함·미상상 케이스 미검출(QA-09 유사입력, NQA-A zero-day 등).
- **self-host GPU vs 외부 API 가정**: QA-01 확장 전략이 갈림 — 팀 결정 선행 필요.
