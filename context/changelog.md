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

<!-- 템플릿
## YYYY-MM-DD — 한 줄 요약
- 변경: <기존> → <신규>
- 사유:
- 영향 ID: DP-0004, artifacts/domain-diagram, ...
-->
