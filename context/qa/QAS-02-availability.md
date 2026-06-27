# QAS-02 운영 안정성 시나리오

> category: QAS | refines: QA-02 (Availability) | updated: 2026-06-24 (round-03 ★ 급간 동기화)

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 인프라 / 노드 **+ 외부 LLM 제공자** |
| **자극 (Stimulus)** | 부분 장애 발생 (노드·컴포넌트 다운) **+ 외부 LLM 제공자 outage / rate-limit(429) 폭주** |
| **대상 (Artifact)** | Workflow 실행 인프라 (long-running·stateful 작업) |
| **환경 (Environment)** | 정상 운영 중 (모델 1건 = 수십 분~수 시간 진행) |
| **응답 (Response)** | 진행 중 작업을 잃지 않고 **멱등 재개**로 서비스 연속성 유지 — 노드 장애는 체크포인트 재개, 외부 LLM 장애는 backoff+큐잉으로 자동 흡수 |
| **응답 측정 (Measure)** | 재기동 ≤4분(★☆☆ 합격 하한) AND in-flight 손실=0, 워크플로우 성공률 ≥99.5%, 외부 LLM 장애 자동 재개율 ≥95%, side-effect 멱등성=100% |

## 비고
- 설계 연결: DP-01(3안 H+Standby — control-plane HA), DP-0003(2안 격리·3안 모니터링 — 외부 장애 흡수).
- 경계: QA-02 = 장애 단위의 **복구**, QA-08 = 타 단위로의 **격리**(blast-radius 차단).
- (★ 급간은 QA-02 등급 척도; round-04 "재기동"=lease 만료→체크포인트 재개 완료로 정의 고정·외부 LLM outage는 자동 재개율로만·lease trade-off 명시 — 급간 수치 불변)
