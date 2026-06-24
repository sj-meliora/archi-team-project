# 변경 이력 (Changelog)

> category: meta
> 최신 pptx를 `../source/`에 넣은 뒤, "무엇이 → 무엇으로, 왜, 영향받는 ID"만 append.
> 매번 전체를 다시 설명하지 않고 델타만 기록한다.

## 2026-06-20 — context 구조화
- `team17_context_snapshot.md`(단일 스냅샷)를 방법론 개념 단위(requirements/qa/dp/artifacts + flat)로 분해.
- 추출 기준: SW_Architect_17조_-_팀_과제_.pptx (45p), 추출 시점 2026-06-19.
- 정합성 메모 6건 → open-issues.md(OI-1~6)로 이관.

## 2026-06-23 — QA 우선순위 재정렬 + QA·QAS 2자리 번호 전환
- 변경: QA 우선순위 재정렬 — QA-01 Scalability / 02 Availability / 03 Controllability / 04 Observability / 05 Efficiency (이전 1위 Efficiency가 5위로). 06~10은 속성 불변, 자릿수만 2자리.
- 변경: QA·QAS ID 포맷 4자리→**2자리**(`QA-0002`→`QA-01`). FR/C/DP는 4자리 유지(추후 전환).
- 사유: 발표 슬라이드·팀 산출물이 참조할 canonical 번호를 팀 우선순위와 일치시키기 위함. 자릿수 혼용 방지로 QA·QAS 10개 일괄 2자리.
- 영향 ID: qa/ 전체(QA·QAS 20파일 리네임), DP-0001~0005, dp4/*, FR-0001~0005, C-0002, overview, INDEX, glossary, open-issues(OI-1).

## 2026-06-24 — QA 디스커션 round-02 반영 (수렴 라운드)
- 변경(QA-03 contention): ② `허용 범위 외 액션 통과 = 0` → **②-1 권한외 액션 차단율**(룰 체커·admission 게이트·"주입→차단/안전정지 latency 분포"·OI-8 무관 현 조건 닫힘) + **②-2 적대적 위반0 acceptance**(OWASP LLM Top-10 매핑 풀세트·NQA-A 채택 OI-8 의존 open-issue로 강등) 2단 분리. "동반 닫힘" 무조건 표현 → 조건부("②-2·NQA-A는 OI-8 채택 시 동반 닫힘; ②-1은 선닫힘") 정정. 헤드라인 ① 유지(교체 불요) + QA-09 비대칭 사유 명시. QAS-03 Response·Measure 동기화. (QA-03 파일 끝 `</content></invoke>` 잔여 아티팩트 제거.)
- 변경(Low cross-link 4건): QA-01 rate-limit 헤드룸 측정단위(계정전역 TPM/RPM·큐별 token-bucket 배분, QA-06 cross-link) / QA-05 캐시 적중률 보조 KPI 노출 / QA-06 외부 rate-limit 계정전역 silent cap 명문화(QA-01 cross-link) / QA-10 CIS p95 컴포넌트 경계 정의 선결 명문화.
- 변경(open-issues): OI-8에 "QA-03 ②-2 ↔ NQA-A 공유 red-team 하네스 채택" 교차 의존 등록 + OI-7에 동일 교차 의존·eval/검증 서브시스템 DP 귀속 갱신.
- 사유: round-02 QA 디스커션(수렴 단계) — actionable [반영]은 QA-03 contention 델타 + Low cross-link뿐. 닫힘 7건(QA-01·02·04·05·06·08·10)은 round-01 반영분 재확인(신규 본문 대수정 없음). NQA-A/B/C 정식 채택·번호 재정렬·예시값 확정은 [이월](OI-8 사람 결정), DP 귀속은 [이월](OI-7 DP 디스커션).
- 영향 ID: QA-03, QAS-03, QA-01, QA-05, QA-06, QA-10, open-issues(OI-7·OI-8).

<!-- 템플릿
## YYYY-MM-DD — 한 줄 요약
- 변경: <기존> → <신규>
- 사유:
- 영향 ID: DP-0004, artifacts/domain-diagram, ...
-->
