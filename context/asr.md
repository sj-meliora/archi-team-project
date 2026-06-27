# ASR — Architecturally Significant Requirements (선정·우선순위 SSoT)

> category: meta | updated: 2026-06-27

## 이 파일의 역할
ASR(아키텍처를 좌우하는 핵심 요구 = **DP 생성 동인**) **선정 목록과 우선순위**의 단일 진실 공급원(SSoT).

- **왜 분리했나**: 종전엔 `QA 번호 = 발표 우선순위`라 우선순위가 바뀌면 QA id를 재번호 → 전 cross-ref·문서가 출렁였다. 이제 **QA id는 안정적 식별자로 고정**하고, **우선순위·ASR 선정은 이 파일에서만** 바꾼다.
- **참조 규칙**: QA/DP 디스커션 스펙·에이전트가 "ASR 목록"이 필요하면 **목록을 하드코딩하지 말고 이 파일을 본다.** 선정·순서가 바뀌면 **여기만** 고치면 된다.

## 현재 ASR (우선순위 순)
| 우선순위 | QA ID (고정 식별자) | 품질속성 | 비고 |
|---|---|---|---|
| 1 | QA-01 | Scalability | |
| 2 | QA-02 | Availability | |
| 3 | QA-03 | Controllability | |
| 4 | QA-04 | Observability | |
| 5 | QA-05 | Efficiency | |
| 6 | QA-07 | Correctness | 우선순위 6위지만 id는 07 (QA-06 공석) |

> **QA-06 공석**: 종전 ASR이던 `QA-06 Security/Safety`는 2026-06-26 제약 `C-03`으로 이관(OI-10) → ASR에서 빠짐. QA id 07~13은 재번호하지 않고 현 위치 유지. **"우선순위" 열이 발표 우선순위이고, "QA ID" 열은 바뀌지 않는 식별자**다.

## 비-ASR QA
ASR이 아닌 QA = **이 표에 없는 QA**(현재 QA-08~13). 건강한·저우선 품질속성으로 DP 생성 동인은 아니다. (DP 디스커션에서 비-ASR 과최적화는 결함으로 본다.)

## 제약(Constraint)과의 관계
`C-03`(보안·안전)은 ASR이 아니라 **제약**이지만 DP-0002/0003을 구동한다 — 모든 대안이 무조건 통과할 pass/fail 게이트라 ATAM 우선순위 비교(★) 대상이 아니어서 ASR 목록엔 없다. (이관 경위는 [open-issues.md](open-issues.md) OI-10.)

## 유지보수 규칙
- **우선순위 변경**: 이 표의 "우선순위" 열만 재정렬. **QA id·파일명은 건드리지 않는다.**
- **ASR 추가/제외**: 이 표에 행을 넣거나 빼고, `open-issues.md`에 결정 근거를 남긴다. **QA 본문·DP 스펙의 목록을 일일이 찾아 고칠 필요 없음**(스펙이 이 파일을 참조하므로).
- 변경 시 `changelog.md`에 델타 append.
