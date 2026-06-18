# GPT ↔ Claude 협업 기반 문서 버전업 Workflow v2.2

---

## 🚀 Quick Start

새 창에서 재개할 때는 아래 4줄만 확인한다.

```text
1. last_status.md 확인
2. Current Master / Current Topic 확인
3. Next Step 확인
4. 이 문서의 해당 Step 절차로 이동
```

| Step | 내용 | 담당 |
|------|------|------|
| S0 | 기준 Master 확인 | 사용자 |
| S1 | GPT와 주제별 논의 | 사용자 + GPT |
| S2 | GPT Delta 작성 및 필요 시 패키지 반영 | GPT |
| S3 | (선택) 사용자 1차 피드백 | 사용자 |
| S4 | Claude: Review Delta + Decision Log 초안 생성 | Claude |
| S5 | 사용자 수용/기각 판단 | 사용자 |
| S6 | Claude: Merge Delta 생성 | Claude |
| S7 | 사용자 최종 승인 | 사용자 |
| S8 | Claude: New Master 생성 또는 스킵 확정 | Claude |
| S9 | Claude: Decision Log 확정 저장 + 패키지 정리 | Claude |

> 세부 의미상 [`S2A`](L1a/readme_workflow.md:22) / [`S2B`](L1a/readme_workflow.md:22) 구분은 유지하되, 운영 문서에서는 기본 흐름을 [`S2`](L1a/readme_workflow.md:22) 하나로 먼저 이해하고 필요 시 하위 절차를 본다.

## Claude Step 헤더 규칙

Claude 담당 Step 시작 시 첫 줄은 아래 형식을 사용한다.

```text
---
## 🔄 [Sn] 단계명 | topic: topicNN | base: vX.XX
---
```

사용자 담당 Step 안내는 아래 형식을 사용한다.

```text
⏳ 다음 Step: [S5] 사용자 수용/기각 판단
```

---


## 🔑 Prompt 설계 핵심 원칙

5.x 구현 프롬프트의 공통 철학, Layer, Artifact, Skill Loading, Environment/Capability Discovery 원칙은 아래 단일 출처를 따른다.

```text
prompt/5_0_common_automation_framework.md
```

이 문서에는 운영 흐름에 직접 필요한 최소 원칙만 유지한다.

- GPT + 사용자는 `무엇을 만들 것인가(What)`를 정한다.
- Roo Code / Claude Code는 현재 환경 기준으로 `어떻게 구현할 것인가(How)`를 구체화한다.
- 개별 5.x 프롬프트는 공통 원칙을 재서술하지 않고 [`5_0_common_automation_framework.md`](L1a/prompt/5_0_common_automation_framework.md) 를 참조한다.

---

## 🔧 구현 프롬프트 운영 보정 사항

Master 문서 5장 구현 항목을 실제 자동화 패키지로 발전시킬 때는 `prompt/` 폴더에 항목별 구현 프롬프트를 둔다.

이때 `prompt` 문서는 MCP 설치/설정 문서가 아니다. `ai-mcp-package-v1.0_workflow_todo` 기반 환경 준비가 이미 선행된 이후의 구현 지시 문서로 사용한다.

```text
기본 전제:
- Roo Code 사용 가능
- Claude Code 사용 가능
- Jira 생성 가능
- Confluence read 가능
- Confluence write/update는 아직 미확인
- Local MCP 기반 사내 연동 환경은 이미 준비된 상태
```

따라서 `prompt` 문서에는 아래 내용을 포함하지 않는다.

```text
제외:
- Node.js 설치 여부 확인
- npm/npx MCP 패키지 설치
- Jira MCP 등록
- Confluence MCP 등록
- claude mcp add 명령어 안내
- .mcp.json 생성
- Roo Code mcp_settings.json 생성
- MCP 서버 설정 절차
- MCP 연결 자체를 위한 초기 환경 구성
```

대신 각 자동화 아이템별 Python 패키지 구현에 필요한 목적, 입력값, 출력값, 연동 대상 시스템, dry-run, 테스트 기준, README 기준을 정의한다.

```text
prompt/
├─ 5_1_jira_feedback_loop_py_package_prompt.md
├─ 5_2_code_analyzer_py_package_prompt.md
├─ 5_3_pre_confluence_child_page_collection_prompt.md
├─ 5_3_weekly_report_collection_py_package_prompt.md
├─ 5_4_hld_code_consistency_check_py_package_prompt.md
├─ 5_5_rca_knowledge_graph_py_package_prompt.md
└─ 5_6_onboarding_knowledge_pack_py_package_prompt.md
```

---

## v2 변경 요약

원본(v1) 대비 다음 네 가지를 수정하였다.

- **GPT Merge → Claude Merge**: 최종 병합 주체를 GPT에서 Claude로 변경하였다. GPT가 병합을 담당하면 Claude 검토 의견이 희석될 위험이 있기 때문이다. Claude는 자신의 검토 기준을 유지한 채 병합을 수행한다.
- **Decision Log 자동 초안**: Decision Log를 별도 Step으로 남기는 대신, Claude가 Review Delta 작성 시 Decision Log 초안을 함께 생성한다. 사용자는 수용/기각 판단만 입력하면 된다.
- **사용자 피드백 2회로 단순화**: 기존 3회 피드백 구조를 S3 선택 피드백, S5 수용/기각 판단, S7 최종 승인 중심으로 단순화하였다. 작은 수정의 경우 S3는 생략 가능하다는 조건도 명시하였다.
- **S2A/S2B 분리**: GPT 단계에서 `S2A GPT Delta 생성`과 `S2B GPT Package Update`를 분리하여, 초안 작성과 실제 패키지 파일 반영 여부를 명확히 관리한다.

---

## 🤖 AI 역할 분담 원칙

```text
GPT 영역:
- S1: 사용자와 주제별 논의
- S2A: GPT Delta 생성
- S2B: 사용자가 요청한 경우 실제 패키지 업데이트 및 ZIP 제공

Claude 영역:
- S4: Review Delta + Decision Log 초안 생성
- S6: Merge Delta 생성
- S8: New Master 생성
- S9: Decision Log 확정 저장

사용자 영역:
- S0: 기준 Master 확인
- S3: 선택 피드백
- S5: 수용/기각 판단
- S7: 최종 승인
```

S2까지가 GPT의 핵심 담당 영역이며, 실제 파일 반영이 필요한 경우 `S2B GPT Package Update`에서 처리한다.

**S2A / S2B 전담 원칙**

S2A(GPT Delta 생성)와 S2B(GPT Package Update)는 **GPT 전용 단계**이다. Claude는 S2A·S2B를 대신 수행하지 않는다.

```text
- S2A GPT Delta 생성        → GPT 전담 (Claude 수행 불가)
- S2B GPT Package Update     → GPT 전담 (Claude 수행 불가)
```

이유: Delta 초안 생성과 패키지 반영은 GPT의 생성 역할에 속하며, Claude가 이를 대신하면 생성(GPT)과 검토·병합(Claude)의 역할 분리가 무너진다. Claude의 담당은 S4(Review)·S6(Merge)·S8(New Master)·S9(Decision Log)에 한정한다.
---

## 1. 목적

이 Workflow의 목적은 대형 Master Document를 GPT와 Claude를 함께 활용하여 안정적으로 버전업하는 것이다.

핵심 목표는 다음과 같다.

- 전체 Master Document를 매번 다시 출력하지 않는다.
- 기준 문서는 Master로 유지한다.
- 변경 내용은 Delta 문서로 작게 관리한다.
- GPT와 Claude의 역할을 분리한다.
- 사용자는 적절한 지점에서 피드백을 제공한다.
- 최종 버전은 병합 후 새로운 Master로 생성한다.
- 버전 증가 기준과 검토 이력을 명확히 유지한다.

이 Workflow는 다음 상황에 적합하다.

- 문서가 커서 한 번에 전체 재작성하기 어렵다.
- GPT와 Claude의 장점을 분리해서 쓰고 싶다.
- 버전업 과정에서 사용자 피드백을 반영하고 싶다.
- 변경 이력과 수용/기각 판단을 추적 가능하게 관리하고 싶다.

---

## 2. 기본 운영 원칙

### 2.1 Master 고정 원칙

기준 문서는 항상 Master Document로 유지한다.

예시:

```text
L1_AI_Automation_Roadmap_v0.34.md
```

이 파일은 현재 기준 문서이며, 모든 변경은 이 파일을 직접 덮어쓰는 방식이 아니라 Delta와 Merge를 거쳐 반영한다.

---

### 2.2 Delta 중심 운영 원칙

변경은 항상 Delta 문서로 관리한다.

예시:

```text
delta/v0.35_topic01_discussion_summary.md
delta/v0.35_topic01_gpt_delta_<topic명>.md
review_logs/v0.35_topic01_claude_review_delta.md
review_logs/v0.35_topic01_merge_delta.md
review_logs/v0.35_topic01_decision_log.md
```

Delta 문서는 전체 문서를 재작성하지 않고, 특정 장 또는 특정 항목에 대한 변경만 다룬다.

---

### 2.3 전체 재작성 금지 원칙

GPT와 Claude 모두 전체 Master Document를 매번 새로 작성하지 않는다.

반드시 다음 중 하나만 수행한다.

- 특정 항목 추가
- 특정 항목 수정
- 특정 항목 삭제 제안
- 검토 의견 작성
- 병합 결과 정리

즉, 전체 문서 재생성이 아니라 Section 단위 Delta 운영을 기본으로 한다.

---

### 2.4 정식 버전 증가 원칙

정식 버전 증가는 최종 병합이 끝난 새 Master 생성 시점에만 수행한다.

예시:

```text
L1_AI_Automation_Roadmap_v0.34.md
↓
GPT Delta
↓
Claude Review Delta + Decision Log 초안
↓
사용자 수용/기각 피드백
↓
Claude Merge Delta
↓
사용자 최종 승인
↓
L1_AI_Automation_Roadmap_v0.35.md
```

중간 산출물은 정식 Master 버전으로 취급하지 않는다.

---

## 3. 역할 분리

### 3.1 GPT 역할

GPT는 확장, 구조화, 초안 작성에 강점을 가진다.

주요 역할은 다음과 같다.

- 아이디어 정리
- 구조화
- 새 장 또는 새 항목 초안 작성
- 용어 통일 제안
- 중복 제거 제안
- GPT Delta 생성

즉, GPT는 만드는 역할을 담당한다.

---

### 3.2 Claude 역할

Claude는 검토, 반박, 병합, Decision Log 생성에 강점을 가진다.

주요 역할은 다음과 같다.

- 논리 충돌 검토
- 빠진 조건 탐지
- 운영 리스크 지적
- 과도한 일반화 검토
- 실제 실행 가능성 평가
- 수정 권고안 제시
- **Review Delta + Decision Log 초안 동시 생성**
- **사용자 피드백 반영 후 최종 Merge Delta 생성**
- **New Master 생성**

즉, Claude는 검토하고 병합하는 역할을 담당한다.

병합을 Claude가 담당하는 이유: GPT가 병합을 수행하면 Claude의 검토 의견이 통합 과정에서 희석될 위험이 있다. Claude가 자신의 검토 기준을 유지한 채 병합하면 검토 품질이 최종 산출물에 그대로 반영된다.

---

### 3.3 사용자 역할

사용자는 방향, 수용/기각 판단, 최종 승인에 집중한다.

주요 역할은 다음과 같다.

- Claude Review Delta 검토 후 수용/기각 판단 입력 (2차 피드백)
- 최종 병합 전 품질 확인 (최종 승인)
- Master에 남길 내용과 Spec로 분리할 내용 결정
- 버전업 승인

사용자는 모든 세부 문장을 직접 작성하는 역할이 아니라, 중요한 의사결정을 제공하는 역할을 맡는다.

---

## 4. 전체 Workflow

전체 흐름은 다음과 같다.

```text
[S0] Current Master 확인
↓
[S1] GPT와 주제별 논의
↓
[S2A] GPT Delta 생성
↓
[S2B] GPT Package Update
↓
[S3] (선택) 사용자 1차 피드백
↓
[S4] Claude: Review Delta + Decision Log 초안 생성
↓
[S5] 사용자 2차 피드백 (수용/기각 판단 입력)
↓
[S6] Claude: Merge Delta 생성
↓
[S7] 사용자 최종 승인
↓
[S8] Claude: New Master 생성
↓
[S9] Decision Log 확정 저장
```

이 구조의 핵심은 다음과 같다.

- Master는 기준점 역할만 한다.
- 변경은 Delta에서 발생한다.
- GPT는 S2A에서 Delta를 생성하고, 필요 시 S2B에서 실제 패키지 파일까지 업데이트한다.
- Claude는 검토, 병합, Decision Log를 담당한다.
- 사용자는 2차 피드백(수용/기각)과 최종 승인 두 지점에서만 판단한다.
- 최종 병합 이후에만 Master 버전이 증가한다.

---

## 5. 상세 절차

### 5.1 [S0] 기준 Master 확인

현재 작업의 기준이 되는 Master Document를 하나 정한다.

예시:

```text
L1_AI_Automation_Roadmap_v0.34.md
```

주의사항:

- 동시에 여러 Master를 기준으로 삼지 않는다.
- 가장 최신 승인본 하나만 기준으로 사용한다.

---

### 5.2 [S1] GPT와 주제별 논의

전체 문서를 한 번에 다루지 않고, 특정 주제 또는 특정 섹션만 대상으로 GPT와 논의한다.

예시 주제:

- Jira Feedback Loop 상세화
- API Call Flow DB 저장 구조
- Weekly Report Collection Failure Scenario
- Common Action Module 분리 기준
- RCA Knowledge Graph 저장 전략

이 단계의 목적은 어떤 변경이 필요한지 정리하는 것이다.

**산출물**

GPT와의 논의 결과를 아래 파일에 저장한다. 새 창에서 재개할 때 S1 맥락을 복원하기 위해 반드시 남긴다.

```text
delta/vX.XX_topicNN_discussion_summary.md
```

포함 내용:

- 논의 주제 및 배경
- 주요 결정 사항
- GPT 제안 방향
- 다음 단계(S2A) 작업 범위

---

### 5.3 [S2A] GPT Delta 생성

GPT는 논의 결과를 바탕으로 Delta 문서를 만든다.

예시 파일명:

```text
delta/vX.XX_topicNN_gpt_delta_<topic명>.md
```

이 문서에는 다음 내용이 포함될 수 있다.

- 대상 장/절
- 추가 문안
- 수정 문안
- 삭제 제안
- 위치 이동 제안
- 용어 변경 제안
- 변경 이유
- 영향 범위

이 단계에서는 전체 Master를 재작성하지 않는다.

**구현 프롬프트 검토 (S2A 역할)**

GPT는 Master Delta와 함께 `prompt/`도 검토하여 다음을 수행한다.

- Master 변경 대상 항목에 대응하는 구현 프롬프트 파일이 있으면 함께 수정 초안을 제안한다.
- 구현 프롬프트 파일이 없는 항목이면 신규 파일 초안을 생성한다.
- 구현 프롬프트 변경 내용은 GPT Delta 문서의 `Proposed Change` 항목에 함께 기술한다.

**S2A 완료 후 `last_status.md` 업데이트**

```text
- Current Topic: (논의한 주제명)
- 진행 중인 Delta 파일: delta/{파일명}_gpt_delta.md
- Last Completed Step: S2A — GPT Delta 생성
- Next Step: S2B — GPT Package Update 또는 S3/S4
```

---

### 5.4 [S2B] GPT Package Update

S2B는 GPT Delta 내용을 실제 패키지 파일에 반영하는 단계이다.

S2B는 항상 수행하는 단계가 아니라, 사용자가 “패키지 업데이트”, “ZIP 제공”, “실제 파일에 반영”을 요청한 경우 수행한다.

수행 내용은 다음과 같다.

- S2A에서 정의한 변경 내용을 대상 파일에 실제 반영한다.
- 필요한 경우 `delta/` 폴더에 GPT Delta 산출물을 저장한다.
- `readme_workflow.md`, `prompt/`, `last_status.md` 등 운영 파일을 업데이트한다.
- 반영 후 ZIP으로 재패키징한다.
- 패키지 파일명은 KST 기준으로 생성한다.

**구현 프롬프트 ↔ Master 동기화 원칙 (S2B 필수)**

`prompt/` 파일을 변경하는 경우, 반드시 Master 본문의 대응 항목도 함께 확인하여 동기화한다.

```text
예시:
  prompt/5_2 의 출력값 변경 (Call Flow JSON → MSC)
  → Master 5.2 Storage/Reuse 항목도 동일하게 반영
```

반대로 Master 본문을 변경하는 경우에도, 대응하는 `prompt/` 파일을 함께 확인한다.
어느 한쪽만 반영하고 나머지를 누락하면 두 문서 간 불일치가 발생한다.

주의사항:

- S2B는 New Master 생성 단계가 아니다.
- S2B에서 Master 본문을 직접 버전업하지 않는다.
- Master 버전 증가는 S8에서만 수행한다.
- S2B 완료 후에도 Claude Review가 필요한 경우 S4로 진행한다.

**S2B 완료 후 `last_status.md` 업데이트**

```text
- Current Topic: (진행 중인 주제명)
- 진행 중인 Delta 파일: delta/{파일명}_gpt_delta.md
- Last Completed Step: S2B — GPT Package Update
- Next Step: S3 (사용자 1차 피드백) 또는 S4 (Claude Review)
```

---

### 5.5 [S3] (선택) 사용자 1차 피드백

이 단계는 선택 사항이다.

큰 구조 변경이나 장 추가처럼 방향 확인이 필요한 경우에만 수행한다.

작은 항목 수정이나 문안 변경의 경우 이 단계를 생략하고 바로 Step 5로 진행한다.

이 단계에서 사용자가 줄 수 있는 피드백 예시는 다음과 같다.

- 이 방향은 맞다
- 이 항목은 너무 상세하다
- 이 내용은 Master가 아니라 Spec로 빼자
- 장 위치를 바꾸자
- 용어를 기존 기준으로 맞추자
- 이 항목은 삭제하자
- 이 항목은 유지하자

**산출물**: 없음. 피드백은 대화로 전달하며 별도 파일을 생성하지 않는다.

---

### 5.5 [S4] Claude: Review Delta + Decision Log 초안 생성

Claude는 Master와 GPT Delta를 기준으로 Review Delta와 Decision Log 초안을 동시에 작성한다.

예시 파일명:

```text
review_logs/vX.XX_topicNN_claude_review_delta.md
```

이 문서의 목적은 다음과 같다.

- 논리 충돌 찾기
- 누락 조건 찾기
- 운영 리스크 찾기
- 중복 지적
- 구현 현실성 검토
- 수정 권고안 제시

Review Delta 권장 마커:

```text
[KEEP]   - 원안 유지 권장
[MODIFY] - 수정 권고 (수정안 포함)
[DELETE] - 삭제 권고
[ADD]    - 추가 권고
[RISK]   - 운영 리스크 지적
[OPEN QUESTION] - 사용자 판단 필요 항목
```

Decision Log 초안은 Review Delta 하단에 함께 포함한다.

Decision Log 초안 형식:

```text
## Decision Log 초안
| 항목 | Claude 권고 | 사용자 판단 | 반영 결과 |
|------|------------|------------|---------|
| 5.3 Precondition 수정 | MODIFY | (입력 대기) | - |
| 5.3 Failure Scenario 추가 | ADD | (입력 대기) | - |
```

사용자는 "(입력 대기)" 칸에 ACCEPT / REJECT / MODIFY 중 하나를 입력한다.

중요한 원칙:

- Claude도 전체 문서를 다시 쓰지 않는다.
- Review Delta만 작성한다.

**구현 프롬프트 정합성 검토 (S4 역할)**

Claude는 Review Delta 작성 시 다음을 추가로 검토한다.

- Master 변경 대상 항목의 구현 프롬프트 파일이 GPT Delta에 포함되었는가.
- 구현 프롬프트 파일이 누락된 경우 Decision Log에 `[ADD]` 항목으로 추가한다.
- 구현 프롬프트 파일이 변경된 경우, Master 본문의 대응 항목(Workflow / Expected Benefit / Storage·Reuse 등)도 동일하게 반영되었는지 교차 확인한다.
- 구현 프롬프트와 Master 본문 간 불일치가 발견된 경우 Decision Log에 `[MODIFY]` 항목으로 추가한다.

---

### 5.6 [S5] 사용자 수용/기각 판단

이 단계가 가장 중요한 사용자 판단 지점이다.

사용자는 Decision Log 초안의 각 항목에 판단을 입력한다.

```text
ACCEPT  - 원안 또는 Claude 권고 그대로 반영
REJECT  - 반영하지 않음 (이유 간략히)
MODIFY  - 절충안 지정 (내용 입력)
DEFER   - 다음 버전으로 미룸
SPEC    - Master가 아닌 Spec 문서로 분리
```

이 단계의 목적은 Claude의 검토 의견 중 실제 반영할 내용을 결정하는 것이다.

---

### 5.7 [S6] Claude: Merge Delta 생성

Claude는 다음 자료를 기반으로 Merge Delta를 만든다.

- Current Master
- GPT Delta
- Claude Review Delta
- 사용자 2차 피드백 (Decision Log 확정)

예시 파일명:

```text
review_logs/vX.XX_topicNN_merge_delta.md
```

이 문서는 최종 반영할 변경을 정리한다.

수행 작업:

- 수용된 변경 반영
- 기각된 변경 제외
- 중복 제거
- 용어 정리
- 위치 정리
- 장 간 정합성 검토

**구현 프롬프트 업데이트 (S6 역할)**

Claude는 Merge Delta 반영과 함께 `prompt/`도 업데이트한다.

- Master 변경이 확정된 항목에 대응하는 구현 프롬프트 파일을 수정한다.
- GPT Delta에서 제안된 구현 프롬프트 초안이 있으면 Claude Review 결과를 반영하여 최종본으로 완성한다.
- 구현 프롬프트 수정 내용은 `last_status.md`에 함께 기록한다.

---

### 5.8 [S7] 사용자 최종 승인

사용자는 병합 결과를 보고 최종 승인 피드백을 준다.

예시:

- 이 버전으로 확정
- 특정 문장만 수정
- 용어만 다시 정리
- 중복 문단 제거
- 이 항목은 다음 버전으로 미루기
- Master에는 넣지 않고 별도 Spec로 분리

이 단계는 최종 품질 확인 단계이다.

---

### 5.9 [S8] Claude: New Master 생성 또는 스킵 확정

최종 승인 후 Claude는 New Master 생성 여부를 확정한다.

```text
master/L1_AI_Automation_Roadmap_vX.XX.md
```

Master 본문 변경이 확정된 경우에만 이 시점에 정식 버전이 올라간다. workflow/prompt/status만 바뀐 topic이면 S8에서 버전업 스킵을 명시한다.

---

### 5.10 [S9] Decision Log 확정 저장 + 패키지 정리

New Master 생성 후 Decision Log를 확정 버전으로 저장한다.

예시 파일명:

```text
review_logs/vX.XX_topicNN_decision_log.md
```

Decision Log 확정본에는 다음이 포함된다.

- 무엇을 반영했는지
- 무엇을 기각했는지
- 왜 그런 판단을 했는지
- Master에 넣은 이유
- Spec로 분리한 이유
- 다음 버전으로 미룬 이유

Decision Log는 같은 논의를 반복하지 않도록 하는 데 중요하다.

Claude가 Step 5에서 초안을 생성하고 Step 6에서 사용자 판단이 입력되므로, Step 10에서는 확정 내용 정리와 패키지 정리만 수행한다.

**패키지 재생성 및 구버전 ZIP 삭제**

S9 완료 후 패키지를 ZIP으로 재생성하며, 이 시점에 **현재 버전 -2에 해당하는 ZIP 파일을 삭제**한다.

**삭제 트리거 조건 (중요)**

ZIP 삭제는 **해당 S9가 New Master 생성(S8)에 뒤따른 경우에만** 수행한다.

```text
- Master 버전업이 있었던 topic (S8 → S9): -2 버전 ZIP 삭제 수행
- Master 버전업이 없었던 topic (prompt/workflow만 수정): ZIP 삭제 트리거 없음
```

이유: "현재 -2 버전"은 Master 버전 기준이므로, Master 버전이 올라가지 않은 S9에서는 -2 기준점이 이동하지 않아 삭제 대상이 새로 생기지 않는다.

**누락분 보정 원칙**

직전까지 Master 버전업 없는 topic이 이어져 ZIP 삭제가 미뤄졌다면, 다음번 New Master 생성 S9에서 **그 시점 기준 -2 이하의 잔여 ZIP을 함께 정리**한다.

```text
예시:
  v0.37 생성 S9에서 v0.35 ZIP 삭제 누락
  → 이후 v0.38 생성 S9에서 v0.35·v0.36 중 -2 이하 잔여분을 함께 삭제
```

```text
예시: 현재 New Master가 v0.37이면
  유지: L1_AI_Automation_*_v0.36_*.zip  (직전 버전)
  삭제: L1_AI_Automation_*_v0.35_*.zip  (현재 -2 버전)
```

단, ZIP 파일명에 Master 버전이 포함되지 않는 경우(날짜시간 기반 파일명) 삭제 대상은 다음 기준으로 판별한다.

```text
- master/ 폴더 내 현재 -2 버전 Master 파일의 생성 시점보다 이전에 만들어진 ZIP
- 단, 가장 최근 2개의 ZIP은 항상 보존한다
```

삭제 이유: ZIP이 누적되면 어느 시점 상태인지 판별이 어려워진다. 직전 버전(-1)은 롤백 가능성을 위해 보존하고, -2 이상은 master/ 폴더의 파일로 복원 가능하므로 삭제한다.

**S9 완료 후 `last_status.md` 업데이트**

```text
- Current Master: master/L1_AI_Automation_Roadmap_v{new_version}.md
- Current Topic: (없음 — 새 topic 시작 필요)
- 진행 중인 Delta 파일: (없음)
- Last Completed Step: S9 — Decision Log 확정 저장
- Next Step: S0 — 새 topic 선정 후 GPT와 주제별 논의 시작
- 메모: v{new_version} 생성 완료. Decision Log: review_logs/v{new_version}_decision_log.md
```

---

## 6. 사용자 피드백 지점

사용자 피드백은 2개 지점이 핵심이다. 1차 피드백은 선택 사항이다.

### 6.1 (선택) GPT Delta 직후

피드백 목적:

- 방향 조정
- 범위 조정
- 장 위치 조정
- 용어 조정

이 단계는 큰 구조 변경 시에만 수행하고, 작은 수정의 경우 생략한다.

---

### 6.2 Claude Review Delta 직후 ← 핵심 판단 지점

피드백 목적:

- Claude 의견 수용/기각 결정
- 절충안 결정
- Master 반영 여부 결정
- Spec 분리 여부 결정
- DEFER 항목 결정

이 단계는 가장 중요한 판단 단계이다. Decision Log 초안의 각 항목에 판단을 입력한다.

---

### 6.3 최종 병합 직전 ← 최종 승인 지점

피드백 목적:

- 최종 품질 확인
- 중복 제거
- 문장 다듬기
- 승인 여부 판단

이 단계는 최종 승인 단계이다.

---

## 7. 파일 구조 권장안

```text
L1a/
├─ master/
├─ delta/
├─ review_logs/
└─ prompt/
```

---

## 7.0 Delta 파일명 규칙

산출물은 작성 주체에 따라 폴더를 분리한다.

**폴더 분리 원칙**

```text
delta/        ← GPT 산출물 (S1 논의 요약, S2A GPT Delta)
review_logs/  ← Claude 산출물 (S4 Review, S6 Merge, S9 Decision Log)
```

**파일명 형식**

```text
vX.XX_topicNN_<type>_<topic명>.md
```

**항목 설명**

```text
vX.XX   : Target Master 버전 (이 산출물이 반영될 버전, 예: v0.37)
topicNN : 해당 버전 내 topic 순번 (예: topic01, topic02)
<type>  : 아래 산출물 유형 중 하나
<topic명>: 논의 주제를 짧게 요약한 snake_case 문자열 (discussion_summary, gpt_delta만 해당)
```

**산출물 유형 및 저장 위치**

| type | 폴더 | Step | 설명 |
|------|------|------|------|
| `discussion_summary` | `delta/` | S1 | GPT와의 논의 요약 |
| `gpt_delta` | `delta/` | S2A | GPT Delta |
| `claude_review_delta` | `review_logs/` | S4 | Claude Review + Decision Log 초안 |
| `merge_delta` | `review_logs/` | S6 | Merge Delta |
| `decision_log` | `review_logs/` | S9 | Decision Log 확정본 |
| `implementation_notes` | `review_logs/` | 구현 중 수시 | 구현 중 발견한 prompt/workflow/schema 개선점, 환경 제약 TODO |

**예시**

```text
delta/v0.37_topic02_discussion_summary.md
delta/v0.37_topic02_gpt_delta_readme_overview_sync.md
review_logs/v0.37_topic02_claude_review_delta.md
review_logs/v0.37_topic02_merge_delta.md
review_logs/v0.37_topic02_decision_log.md
```

> **혼동 주의**: 파일명의 `vX.XX`는 Base Master 버전이 아니라 **Target Master 버전**이다.
> Target 버전은 "해당 topic 작업이 완료됐을 때 실제 도달한 Master 버전"을 의미한다.
> Master 버전업이 발생한 경우 → 새 버전 번호 사용 (예: Base v0.36 → New Master v0.37이면 `v0.37_*`)
> Master 버전업이 없는 경우 → Base 버전과 동일하게 사용 (예: Base v0.36 유지이면 `v0.36_*`)

---

## 7.1 패키지 전달 파일명 규칙

패키지를 zip으로 전달할 때는 파일명 끝에 날짜시간을 추가한다.

**형식**

```text
L1_AI_Automation_YYYYMMDD_HHMM.zip
```

**예시**

```text
L1_AI_Automation_20260614_1530.zip
```

날짜시간은 zip 생성 시점 기준 로컬 시간을 사용한다.

이 규칙의 목적은 다음과 같다.

- 여러 버전의 zip이 쌓일 때 구분 가능하다.
- 어느 시점 상태의 패키지인지 추적 가능하다.
- 파일명 충돌을 방지한다.

---

## 8. Delta 문서 템플릿

### 8.1 GPT Delta 템플릿

```md
# Delta Metadata
- Base Version:
- Target Version:
- Author AI: GPT
- Topic:
- Date:

# Change Type
- [ ] Add
- [ ] Modify
- [ ] Delete
- [ ] Move

# Target Section
- 대상 장/절

# Proposed Change
- 실제 반영 문안

# Rationale
- 변경 이유

# Impact
- 용어 영향
- 다른 장 영향
- 구현 우선순위 영향

# Merge Instruction
- insert / replace / append / reorganize
```

---

### 8.2 Claude Review Delta 템플릿

```md
# Review Delta Metadata
- Base Version:
- GPT Delta:
- Author AI: Claude
- Date:

# Review Items

## [KEEP] 항목명
- 판단 근거

## [MODIFY] 항목명
- 현재 문안:
- 수정 권고안:
- 수정 이유:

## [DELETE] 항목명
- 삭제 이유:

## [ADD] 항목명
- 추가 권고 문안:
- 추가 이유:

## [RISK] 항목명
- 리스크 내용:
- 완화 방안:

## [OPEN QUESTION] 항목명
- 질문 내용:
- 판단 필요 이유:

---

# Decision Log 초안

| 항목 | Claude 권고 | 사용자 판단 | 반영 결과 |
|------|------------|------------|---------|
|      |            | (입력 대기) |         |
```

---

## 9. Claude 검토 체크리스트

Claude 검토 품질을 일정하게 유지하려면 검토 기준을 고정하는 것이 좋다.

권장 체크리스트는 다음과 같다.

1. 기존 용어 체계와 충돌하는가
2. Knowledge Layer 구조와 연결되는가
3. Closed-loop 구조와 정합성이 있는가
4. Action / Verify / Feedback 단계가 빠졌는가
5. 실제 운영 가능한 수준인가
6. 다른 장과 중복되는가
7. Failure Scenario가 빠졌는가
8. Human-in-the-loop 필요 지점이 명확한가
9. 구현 우선순위가 비현실적이지 않은가
10. Master에 둘 내용인지 Spec로 분리할 내용인지 구분이 필요한가
11. Master 변경 대상 항목에 대응하는 구현 프롬프트 파일이 GPT Delta에 포함되었는가

---

## 10. 권장 운영 방식

### 방식 A. 섹션 단위 왕복 (권장)

한 번에 전체 문서를 다루지 않고 섹션 단위로 작업한다.

예시:

```text
5.1 Jira Feedback Loop만 논의
→ GPT Delta 작성
→ (선택) 사용자 1차 피드백
→ Claude Review Delta + Decision Log 초안 작성
→ 사용자 수용/기각 판단 입력
→ Claude Merge Delta 작성
→ 최종 승인
→ Claude: Master 반영
```

이 방식은 품질이 높고 충돌이 적다.

---

### 방식 B. 장 단위 왕복

문서 구조가 안정적일 때는 장 단위로 작업할 수 있다.

예시:

```text
5장 전체
8장 전체
10장 전체
```

다만 문서가 커질수록 장 단위보다는 섹션 단위가 더 안정적이다.

---

### 방식 C. Spec 분리 병행

상세 구현 내용이 길어질 경우 Master에 모두 넣지 않고 Spec 문서로 분리한다.

예시:

- Master에는 개요와 원칙만 유지
- 상세 Workflow, 데이터 스키마, 프롬프트, Failure Scenario는 Spec 문서로 분리

이 방식은 Master Document 비대화를 막는 데 유리하다.

---

## 11. 비권장 방식

### 11.1 전체 문서를 매번 GPT와 Claude가 번갈아 재작성하는 방식

예시:

```text
GPT가 v0.35 전체 생성
→ Claude가 v0.36 전체 생성
→ GPT가 v0.37 전체 생성
```

문제점:

- 버전 증가 기준이 흐려진다
- 중복과 충돌이 많아진다
- 변경 주체 추적이 어렵다
- 사용자 피드백 시점이 모호해진다

---

### 11.2 검토 결과를 기록 없이 반영하는 방식

문제점:

- 왜 반영했는지 남지 않는다
- 같은 논의를 반복하게 된다
- 기각 사유가 사라진다

Decision Log를 반드시 남겨야 한다.

---

### 11.3 여러 Master를 동시에 기준으로 삼는 방식

문제점:

- 기준 문서가 흔들린다
- 병합 충돌이 커진다
- 버전 추적이 어려워진다

항상 최신 승인본 하나만 Master로 사용해야 한다.

---

### 11.4 GPT가 최종 Merge를 담당하는 방식 (v1에서 변경)

문제점:

- Claude 검토 의견이 통합 과정에서 희석될 위험이 있다
- GPT는 생성 지향이라 비판적 내용을 자연스럽게 완화하는 경향이 있다
- 검토 품질이 최종 산출물에 반영되지 않는다

Claude가 검토와 병합을 함께 담당해야 검토 기준이 최종 Master에 유지된다.

---

## 12. 최종 정리

이 Workflow의 핵심은 다음 한 문장으로 정리할 수 있다.

```text
전체 문서를 AI끼리 번갈아 다시 쓰지 말고, 기준 Master는 고정하고 Delta만 왕복시킨다.
```

운영 핵심은 다음과 같다.

- Master는 고정한다.
- GPT는 초안을 만든다.
- Claude는 검토하고 병합하며 Decision Log 초안을 생성한다.
- 사용자는 수용/기각 판단(2차 피드백)과 최종 승인 두 지점에서만 판단한다.
- 최종 병합 후에만 새 Master를 만든다.
- 결정 결과는 Decision Log로 남긴다.

이 구조를 따르면 대형 문서도 안정적으로 버전업할 수 있고, GPT와 Claude의 장점을 동시에 활용할 수 있다.
