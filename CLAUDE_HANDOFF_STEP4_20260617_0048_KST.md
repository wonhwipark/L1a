# Claude Handoff — Step 4 Review Package

- Generated: 2026-06-17 00:48 KST
- Target Step: S4 — Claude Review Delta + Decision Log Draft
- Base Package: `L1_AI_Automation_20260617_0044_KST.zip`
- Updated Package Purpose: Step 4 진입용 정리본

---

## 1. Step 4 Entry Header

Claude Code는 Step 4 시작 시 아래 헤더를 먼저 출력한다.

```text
---
## 🔄 [S4] Claude Review Delta | topic: topic04 | base: v0.37/v0.38-candidate
---
```

---

## 2. Current Package Structure

현재 5.x 자동화 프롬프트 위치는 다음과 같다.

```text
prompt/
```

이전 명칭 변경 이력:

```text
automation_prompt/ → skills/ → prompt/
```

최종 채택 명칭:

```text
prompt/
```

---

## 3. Primary Review Targets

Step 4에서 Claude가 우선 검토할 GPT Delta는 다음이다.

```text
gpt_delta/v0.39_topic04_gpt_delta_5_0_1_confluence_child_discovery_strategy_20260617_0037_KST.md
gpt_delta/v0.41_topic04_gpt_delta_skills_to_prompt_20260617_0044_KST.md
```

참고:

```text
gpt_delta/v0.40_topic04_gpt_delta_automation_prompt_to_skills_20260617_0040_KST.md
```

`v0.40`은 중간 변경 이력이며, 최종 구조는 `v0.41`의 `prompt/` 기준을 따른다.

---

## 4. User Decisions Already Confirmed

### 4.1 Final Goal

최종 목표는 prompt 문서 자체가 아니라 Python 자동화 도구이다.

```text
Prompt 정리
↓
Python package 구현
↓
CLI 제공
↓
필요 시 npm/npx wrapper 제공
```

---

### 4.2 5.3 Weekly Report Runtime Assumptions

```text
- Confluence MCP + REST API 사용 가능
- 매주 Parent Page link는 사용자가 제공
- 매주 Previous Week Page link도 사용자가 제공
- 팀원 목록은 사용하지 않음
- missing member detection은 하지 않음
- 발견된 Child Page 내용만 사용
- Previous Week Page 양식에 맞춰 이번 주 TL Weekly Report 초안 생성
```

---

### 4.3 5.0.1 Decision

`5.0.1 Confluence Child Page Discovery Strategy`를 신규 prompt로 추가하는 방향이다.

역할:

```text
Parent Page link를 입력받아 Child Page를 찾는 여러 방식을 시도하고,
성공한 방식만 strategy profile로 저장한다.
```

중요:

```text
Parent Page ID, Parent Page URL, Child Page URL 목록, Child Page ID 목록은 저장하지 않는다.
```

이유:

```text
주간보고 Parent Page는 매주 새로 생성되기 때문이다.
```

---

### 4.4 Artifact Filename Rule

모든 output artifact 파일명 끝에는 KST 저장 날짜/시간을 포함한다.

형식:

```text
<base_filename>_<YYYYMMDD>_<HHMM>_KST.<ext>
```

예:

```text
weekly_report_draft_20260617_0045_KST.md
child_page_discovery_profile_20260617_0045_KST.json
```

---

## 5. Expected Step 4 Output

Claude는 다음 파일을 생성하거나 제안한다.

```text
review_logs/v0.41_topic04_claude_review_delta_<YYYYMMDD>_<HHMM>_KST.md
```

Decision Log 초안은 필요 시 다음 위치에 생성한다.

```text
review_logs/v0.41_topic04_decision_log_draft_<YYYYMMDD>_<HHMM>_KST.md
```

---

## 6. Review Questions for Claude

Claude는 아래 항목을 중점 검토한다.

```text
1. prompt/ 폴더명이 현재 목적에 적합한가?
2. 5.0.1을 5.3에서 분리하는 구조가 타당한가?
3. 5.0.1 profile이 특정 주차 데이터를 저장하지 않는 설계가 맞는가?
4. 5.3이 latest discovery profile을 찾는 방식 또는 사용자 지정 profile을 받는 방식이 안전한가?
5. 모든 artifact timestamp 규칙이 일관되게 적용되어 있는가?
6. 현재 패키지에서 prompt 경로 참조가 누락 없이 동기화되었는가?
```

---

## 7. Next Step After Claude Review

Claude Review 이후 사용자는 S5에서 각 항목을 판단한다.

```text
ACCEPT
REJECT
MODIFY
DEFER
```
