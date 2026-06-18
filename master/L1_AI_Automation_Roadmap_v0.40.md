# L1 AI Automation Roadmap v0.40
---
## Document Purpose
이 문서는 L1 AI Automation Roadmap의 v0.40 Master Document이다.
이 문서 하나만으로 새 창에서도 v0.41, v1.0으로 계속 버전업할 수 있도록 독립형 문서로 구성한다.
버전업 운영 절차(GPT ↔ Claude 협업 기반 Delta Workflow)는 `readme_workflow.md`를 참조한다.
문서 작성 원칙은 다음과 같다.
- 축약 금지; 생략 금지; 완전 독립형 문서; Knowledge Layer 기준 용어 통일; 자동화 Workflow와 Knowledge Architecture 연결; L1 업무 자동화 Master Document 기준 유지
---
## 1. Background
초기 AI 활용은 Code Review, HLD 작성, API 분석 등 개별 업무를 지원하는 수준이었다.
실제 업무 적용 결과, 단순히 AI를 사용하는 것보다 사람이 반복적으로 수행하는 판단과 추적 업무 자체를 자동화하는 것이 훨씬 높은 ROI를 가진다는 것을 확인하였다.
특히 다음 영역에서 효과가 크다.
- Critical Defect; API Call Flow 분석; HLD 관리; Confluence 주간보고; Jira 관리; RCA 재활용; 조직 Knowledge 축적
향후 목표는 Prompt Engineering이나 단순 Skill 추가가 아니라,
**Closed-loop 기반의 L1 업무 자동화 체계 구축**
이다.
L1 업무에서 AI의 가치는 단순 문서 작성보다 반복 판단의 자동화에서 더 크게 발생한다.
Critical Defect인지 판단하고, API Call Flow를 추적하고, HLD와 Code의 Gap을 확인하고, Jira Close 이후 재발 방지 Rule을 남기는 작업은 모두 반복 판단 업무이다.
따라서 본 문서는 L1 업무를 개별 AI 활용이 아니라 Closed-loop 기반의 Engineering Operation 자동화로 발전시키기 위한 기준 문서이다.
---
## 2. Automation Item Discovery Principle
자동화 대상은 다음 질문을 기준으로 선정한다.
```text
사람이 반복적으로 판단하고 있는 업무는 무엇인가?
```
특히 다음 유형은 높은 ROI를 가진다.
- Checking; Comparison; Classification; Tracking; Prioritization; Gap Detection; Prevention
Checking은 특정 조건이 만족되는지 반복적으로 확인하는 업무이다.
예시는 Critical Defect Rule 위반 여부, HLD 필수 항목 누락 여부, Jira 필수 필드 입력 여부, 주간보고 작성 여부 확인이다.
Comparison은 두 개 이상의 산출물을 비교하여 차이를 찾는 업무이다.
예시는 HLD와 Code 비교, Jira 설명과 실제 Patch 비교, 기존 RCA와 신규 Issue 비교, MSC와 실제 Call Flow 비교이다.
Classification은 입력 데이터를 정해진 기준에 따라 분류하는 업무이다.
예시는 Defect 유형 분류, Jira 이슈 심각도 분류, Risk 유형 분류, Root Cause 유형 분류이다.
Tracking은 시간에 따라 상태 변화를 추적하는 업무이다.
예시는 Jira 상태 변화 추적, Review 상태 추적, 주간보고 Risk Trend 추적, HLD 변경 이력 추적, Defect 재발 여부 추적이다.
Prioritization은 여러 항목 중 우선순위를 결정하는 업무이다.
예시는 Critical Defect 우선순위, Review 대상 우선순위, Jira 대응 우선순위, 주간보고 Risk 우선순위 결정이다.
Gap Detection은 기대 상태와 실제 상태의 차이를 찾는 업무이다.
예시는 HLD에는 있는데 Code에는 없는 Branch 검출, Code에는 있는데 HLD에 없는 Behavior 검출, Jira에 언급된 Fix가 실제 Patch에 없는 경우 검출이다.
Prevention은 과거 이슈가 재발하지 않도록 사전에 차단하는 업무이다.
예시는 RCA 기반 Prevent Rule 생성, Critical Defect Rule 강화, False Positive History 반영, Code Review Rule 업데이트, TC 추가 제안이다.
---
## 3. Closed-loop Structure
모든 Workflow는 다음 구조를 지향한다.
```text
Input
  ↓
Analyze
  ↓
Decision
  ↓
Action
  ↓
Verify
  ↓
Feedback
  ↓
Knowledge Layer
  ↓
Reuse
```
Input은 Branch, Code, API, HLD, Jira, Log, Confluence Page, Weekly Report, TC Result, Review Comment가 될 수 있다.
Analyze는 Code Path 분석, API Call Flow 분석, Defect Pattern 분석, HLD 구조 분석, Jira 내용 분석, RCA Root Cause 분석, Risk 문장 추출을 수행한다.
Decision은 Defect 여부, Critical 여부, Fix 필요 여부, Jira 생성 여부, HLD 업데이트 필요 여부, Rule 업데이트 필요 여부를 판단한다.
Action은 P4 Shelve 생성, Jira 등록, Jira 업데이트, Confluence Page 생성, Confluence Page 업데이트, Mail 초안 생성, Teams Notification, Calendar Meeting 생성, Dashboard 업데이트를 수행한다.
Verify는 Patch 정상 생성, P4 Shelve 정상 등록, Jira Link 생성, Confluence Page 업데이트, Mail 수신자, TC 추가 여부를 확인한다.
Feedback은 False Positive 등록, Rule 수정, Prompt 수정, Skill 수정, Workflow 개선, Action 기준 조정을 수행한다.
Knowledge Layer는 Workflow 결과가 저장되는 구조화된 지식 계층이며 LLM 자체가 아니다.
Knowledge Layer는 Rule DB, API Call Flow DB, RCA Knowledge Graph, Prompt / Skill DB, Report DB, Team Knowledge를 포함한다.
Reuse는 축적된 지식을 다음 업무에 다시 사용하는 단계이다.
---
## 4. Existing Automation
### 4.1 Critical Defect Workflow
현재 구축된 Workflow
```text
Branch
  ↓
Folder Scan
  ↓
Critical Defect Detect
  ↓
Fix Proposal
  ↓
Patch 생성
  ↓
P4 Shelve
  ↓
Jira 등록
```
향후 확장
```text
Detect
  ↓
Fix
  ↓
Shelve
  ↓
Jira
  ↓
Feedback
  ↓
Rule Update
  ↓
다음 Branch Scan
```
---
## 5.0 Common Automation Framework

모든 5.x 자동화 항목은 다음 공통 원칙을 따른다. 상세는 [`prompt/5_0_common_automation_framework.md`](L1a/prompt/5_0_common_automation_framework.md)을 참조한다.

### 5.0.5 Prefer Existing Environment

새로운 도구나 패키지를 추가하기보다 기존 환경의 bash/grep/find 등 도구를 최대한 활용한다.
예: Track B (Code Analyzer)는 Python CLI가 아닌 bash 기반 정적 추출 사용.

### 5.0.9 Skill Layer vs Workflow Layer

**Skill Layer:** 재사용 가능한 분석 로직 (staged-code-analyzer, staged-code-review 등)을 스킬 형태로 구현.
Roo Code/Claude Code로 실행하며, 새 L1a 버전마다 07_skills_vX.X.zip 형태로 배포.

**Workflow Layer:** Skill을 호출하는 실행 프롬프트를 `L1a/prompt/` 폴더에 저장.
팀원이 프롬프트를 직접 Claude Code/Roo Code에 입력하여 실행.

두 Layer는 명확히 분리 관리하며, Skill 변경 시 프롬프트와의 호환성을 확인한다.

### 5.0.10 Staged Analysis for Large Codebases

대용량 폴더(30분 이상 분석)는 다음 단계로 나눈다:

1. **Phase 0:** 전체 인벤토리 (빠른 카운트)
2. **Phase 1..N:** 단계별 REQ-site 분석 (각 Stage는 독립적 문맥)
3. **Checkpoint:** 각 Stage 완료 후 `review_progress_<YYYYMMDD>_<HHMM>_KST.md` 저장
4. **Global Carry Mechanism:** Stage별 분석 결과 중 공유 요소(예: IPC CNF 파일 처리)는 1회만 분석하고 이후 Stage에서 재사용
5. **Phase F:** 전 Stage 결과 통합 및 최종 산출물 생성

이를 통해 문맥 초과 위험을 방지하고, 재개/재실행 시 효율성을 높인다.

---
## 5. Key Automation Items
### 5.1 Jira Feedback Loop
```text
(1) Critical Defect Detection
      ↓
(2) AI Root Cause Analysis
      ↓
(3) AI Patch Proposal
      ↓
(4) P4 Shelve Creation
      ↓
(5) Jira Creation
    (Shelve Number + Root Cause Summary + Patch Proposal Summary 포함)
      ↓
(6) Review & Discussion
      ↓
(7) Resolution Decision
    ├─ Reject
    │     ↓
    │   False Positive Rule 정리
    │     ↓
    │   Skill Update
    └─ Resolve
          ↓
        Prevent Rule 정리
          ↓
        Skill Update
```
Jira Feedback Loop는 Critical Defect Workflow의 후속 단계이다.
AI가 결함 후보의 Root Cause를 분석하고 Patch를 제안한 후 P4 Shelve를 생성한다.
Jira 등록 시 Shelve Number, Root Cause Summary, Patch Proposal Summary를 포함하여 Reviewer가 즉시 수정 후보를 확인할 수 있도록 한다.
Review 결과는 Reject와 Resolve로 분기한다. Reject 사례는 False Positive Rule로, Resolve 사례는 Prevent Rule로 정리하여 각각 Skill에 반영한다.
Jira Close 상태는 Jira MCP 폴링(JQL)으로 감지하고 Resolution 필드로 Reject/Resolve를 분류한다.
목적은 Jira 처리 결과를 Rule, Skill, Prompt, Workflow 개선으로 연결하는 것이다.
**Expected Benefit**
- False Positive 감소; 경험 축적; Rule 강화; Review 결과 재사용; Critical Defect 검출 정확도 향상; Jira Close 이후 지식 손실 방지
**Precondition**
- 4.1 Critical Defect Workflow 완성 필요; Jira MCP 연동 필요; Resolution 필드 기준 정의 필요 (Reject/Resolve 분류 기준); Review Comment 수집 방식 필요; Perforce MCP 또는 p4 CLI 연동 필요
**Storage / Reuse**
- Rule DB; False Positive History; Prevent Rule; Review Feedback; Rule Version; Prompt / Skill DB; 다음 Branch Scan에서 Prevent Rule 재사용; 유사 Defect 검출 시 기존 Jira와 RCA 연결
---
### 5.2 Code Analyzer

Code Analyzer는 HLD 없이 코드만 존재하는 기존 구현을 분석하여 **Call Flow의 가시화(Call Graph + MSC) 및 간단한 모듈 설명 문서**를 생성하는 Workflow이다.

특히 대용량 폴더(30분 이상의 분석 시간)를 처리할 때, §5.0.10 Staged Analysis 메커니즘을 사용하여 분석을 단계별로 나누고, Global CNF Carry를 통해 CNF 파일 분석 결과를 모든 단계에서 재사용한다.

#### 5.2.1 실행 옵션 (Execution Tracks)

Code Analyzer는 두 가지 Track으로 운영된다:

**Track A: Skill-based Staged Analysis (권장 — 메인 경로)**

```text
코드 루트 경로 + 분석 대상 폴더 + 단계 범위 지정
  ↓
[Phase 0] 전체 폴더 인벤토리 (함수/클래스/호출 계수)
  ↓
[Phase 1..N] 단계별 분석 (각 단계별 REQ-site 파일 그룹 분석)
  │  각 단계:
  │  - 대상 폴더 코드 구조 분석 (함수/클래스/호출 관계/Branch 조건)
  │  - Call Flow 추출 (IPC REQ 사이트 중심)
  │  - Global CNF Carry 체크포인트 적용 (CNF 처리 파일 재사용)
  │  - review_progress_<YYYYMMDD>_<HHMM>_KST.md 저장
  │
[Phase F] 전 Stage 결과 통합
  ↓
Call Flow MSC (PlantUML) 생성
  ↓
간단한 모듈 설명 문서 + JSON 메타데이터
  ↓
(선택) Confluence 업로드 또는 로컬 저장
```

**특징:**
- `staged-code-analyzer` Skill로 구현 (§5.0.9 Skill Layer)
- `5_2_code_analyzer_track_a_prompt.md`로 실행
- 대용량 폴더(30분+) 분석 시 문맥 초과 위험 방지
- Global CNF Carry: CNF 파일(IPC 응답)은 모든 Stage에서 동일하므로, 1회 분석 후 다음 Stage에서 재사용

**Track B: Static Extraction (선택적 — 사전 단계)**

```text
코드 루트 경로 지정
  ↓
Bash 프롬프트 기반 정적 구조 추출
  (find / wc / ctags / grep 등 기존 환경 활용)
  ↓
구조 JSON 생성 (module layout, function inventory, include hierarchy)
  ↓
Track A 입력으로 활용 (선택적)
```

**특징:**
- §5.0.5 "Prefer Existing Environment" 원칙 준수
- Python 패키지 불필요; bash 프롬프트 기반 실행
- `5_2_code_analyzer_track_b_prompt.md`로 실행
- Track A 진입 전 "코드 구조 전체 파악" 필요 시 사용 (선택적)
- 시간 효율: 구조 추출만 ~5분, HLD 산문 생성은 미단축

#### 5.2.2 Global CNF Carry 메커니즘

L1 모뎀 구현에서 IPC REQ/CNF는 다음과 같은 구조를 가진다:

- **IPC REQ 사이트:** 여러 파일에서 발생 (브랜치별로 다름)
- **IPC CNF 처리:** 특정 1개 파일에서만 처리 (모든 REQ의 응답 처리)

따라서 대용량 폴더를 Stage로 분할할 때:
1. **[Phase 1]** CNF 파일 포함 Stage에서 CNF 처리 로직을 완전히 분석 → review_progress_<YYYYMMDD>_<HHMM>_KST.md 체크포인트에 저장
2. **[Phase 2..N]** 이후 Stage에서는 CNF 분석 결과를 재사용 (CNF 파일 재분석 불필요)
3. **[Phase F]** 모든 Stage의 REQ + 공유 CNF 결과 통합 → 최종 Call Flow MSC 생성

#### 5.2.3 간단한 모듈 설명 문서 (Module Overview)

각 모듈당 50~100줄 분량의 Markdown 문서:
- **모듈 개요** (2~3줄): 모듈의 역할 한 문단
- **주요 책임** (bullet 2~3개): "REQ 처리", "CNF 응답", "타이밍 관리" 등
- **주요 함수/API** (리스트): 함수명 + 1줄 설명
- **주요 IPC 호출** (옵션): REQ/CNF 구분, 호출 순서

#### 5.2.4 Expected Benefit

- Call Flow 가시화 (MSC): 대용량 코드에서도 IPC 호출 흐름 시각화
- 구현 의도 복원: 간단한 모듈 설명으로 신규 인원 빠른 이해
- 문맥 초과 방지: Staged Analysis로 30분+ 분석도 완료
- 코드 리뷰 품질: Call Flow 이해 → 리뷰 포인트 명확화
- 후속 항목 입력: 5.4/5.5/5.6의 기반 자료 제공

#### 5.2.5 Precondition

- 분석 대상 코드 루트 경로, 단계별 폴더 범위(예: Phase 1: TxSwitchMngr/, Phase 2: TxCfgMngr/), 분석 대상 파일 확장자(.c, .h 등) 사전 확정 필요
- Track A 사용 시: Claude Code 또는 Roo Code 사용 가능
- Track B 사용 시: bash 환경 + find/wc/ctags/grep 기본 도구

#### 5.2.6 Storage / Reuse

- Call Flow MSC (PlantUML) — 5.4 Gap Detection, 5.5 RCA Knowledge Graph에 입력으로 재사용 가능
- 간단한 모듈 설명 문서 — 신규 인원 온보딩, 코드 리뷰 가이드, 5.6 Onboarding Knowledge Pack의 기반
- 구조 분석 메타데이터 (JSON) — 5.4 Consistency Check, 5.5 RCA Knowledge Graph의 API 연결 정보로 재사용 가능
- review_progress.md (Stage 체크포인트) — 재개 또는 다시 실행 시 Skip 지점 명시

---
### 5.3-pre Confluence Child Page Collection
```text
Parent Page URL 입력
  ↓
Child Page 탐색 (REST API → MCP → 본문 링크 추출 → Label → Title → Fuzzy → User Assisted)
  ↓
Child Page 본문 취합
  ↓
취합 원본 draft 저장
  (%USERPROFILE%\artifacts\weekly_report_draft_<YYYYMMDD>_<HHMM>_KST.md)
```
Confluence Child Page Collection은 5.3 Weekly Report Collection의 선행 실행 항목이다.
매주 새로 생성되는 Parent Page URL을 기준으로 실제 Child Page를 탐색하고, 발견된 Child Page 본문을 취합하여 5.3의 입력 draft를 생성한다.
탐색에 성공한 strategy는 선택적으로 profile로 저장하여 다음 주 실행 시 재사용할 수 있다.
현재 AI 실행 프롬프트 형태로 운용하며, 추후 Python 패키지로 구현 예정이다.
**Expected Benefit**
- 매주 반복되는 Child Page 탐색 및 본문 취합 자동화; 탐색 실패 시 단계별 fallback 제공; 5.3 실행을 위한 입력 draft 안정적 생성
**Precondition**
- Confluence read 가능 (MCP 또는 REST API); Parent Page URL을 사용자가 매주 제공
**Storage / Reuse**
- Child Page 탐색 strategy; 취합 draft; Confluence Page ID; 다음 주 Parent Page 탐색 시 strategy 재사용; Confluence MCP 기반 다른 자동화의 page 탐색 baseline
---
### 5.3 Weekly Report Collection
```text
5.3-pre draft (매주 새로운 Child Page 본문)
  ↓
이전 주 양식 기반 필드 파악 (Risk, Issue, Action Item 등)
  ↓
필드별 내용 추출
  ↓
이전 주 양식 복제 및 필드 갱신
  ↓
Confluence 업로드 또는 로컬 저장
```
Weekly Report Collection은 매주 반복되는 주간보고 작성을 자동화하는 Workflow이다.
5.3-pre에서 생성된 Child Page draft를 입력받아, 이전 주 양식을 기준으로 필드를 파악하고, 이번 주 내용으로 채운다.
Risk Trend, Issue Tracking, Action Item 추적을 자동으로 전월대비 비교하고 우선순위를 제안한다.
**Expected Benefit**
- 주간보고 작성 시간 단축; 주간 반복 질문 자동화; Risk/Issue 추적 강화; Action Item 완료도 추적; 팀 커뮤니케이션 개선
**Precondition**
- 5.3-pre Confluence Child Page Collection 완성 필요; 이전 주 주간보고 양식 존재 필요; Confluence Update 권한 필요; Risk 분류 기준 필요; Action Item 상태 기준 필요
**Storage / Reuse**
- Weekly Report; Risk Trend; Issue List; Action Item; Confluence Link; Next Week 계획; 주간보고 이력; Risk Pattern 축적; Action Item 재사용
---
### 5.4 HLD ↔ Code Consistency Check
```text
HLD
  ↓
Code 분석
  ↓
Gap Detection
  ↓
누락 Branch 발견
  ↓
HLD 수정안
  ↓
Confluence Update
```
HLD ↔ Code Consistency Check는 HLD 문서와 실제 Code 구현이 일치하는지 확인하는 Workflow이다.
5.2에서 생성된 Call Flow MSC와 JSON 메타데이터를 입력받아, 기존 HLD의 Sequence Diagram 및 구현 설명과 비교하여
runtime Call Flow 변경 여부, 누락된 Branch, 구현 의도 불일치를 검출한다.
**Expected Benefit**
- 설계 최신화; 문서 품질 향상; Code와 HLD 간 불일치 감소; Review 품질 향상; 신규 인원 이해도 향상; 기능 변경 이력 추적 강화
**Precondition**
- HLD 형식 표준화 필요; 5.2 Code Analyzer (Track A) 완료 권장; Confluence Update 권한 필요; Code 분석 범위 지정 필요. 주의: 5.2 Track B 단독 완료는 불충분 (Call Flow MSC 필수)
**Storage / Reuse**
- HLD Section; Code Path; API; Branch Condition; Gap Type; Suggested HLD Update; Confluence Link; Review Status; 다음 HLD 작성 시 Gap Pattern 재사용; Code Review 시 HLD 누락 검출
---
### 5.5 RCA Knowledge Graph
```text
Issue
  ↓
Log
  ↓
Root Cause
  ↓
Fix
  ↓
Jira
  ↓
HLD
  ↓
TC
  ↓
API
  ↓
Knowledge Layer
```
RCA Knowledge Graph는 Issue, Log, Root Cause, Fix, Jira, HLD, TC, API를 연결하는 구조화된 지식 그래프이다.
RCA를 Graph 형태로 저장하면 특정 API, 특정 Root Cause, 특정 Log Pattern, 특정 Jira, 특정 TC를 기준으로 과거 사례를 재사용할 수 있다.
**Expected Benefit**
- 분석 시간 감소; 경험 재사용; 신규 인원 지원; 유사 Issue 검색; Root Cause Pattern 축적; Prevent Rule 생성; TC 보강 근거 확보
**Precondition**
- 5.2 Code Analyzer (Track A) 구축 필수. 주의: 5.2 Track B 단독 완료는 불충분 (API 연결 정보가 Track B JSON에 불완전).
- Jira와 RCA 문서 연결 필요; Log Pattern 저장 기준 필요; Root Cause 분류 기준 필요
**Storage / Reuse**
- Issue; Log; Root Cause; Fix; Jira; HLD; TC; API; Known Defect; Prevent Rule; 신규 Issue 분석 시 유사 RCA 검색; Critical Defect Rule 생성
---
### 5.6 Onboarding Knowledge Pack 자동 생성
```text
도메인 태그 + 신규 인원 ID
  ↓
API Call Flow DB 조회
  ↓
RCA Knowledge Graph 조회
  ↓
Call Flow MSC + 모듈소개 연결
  ↓
Onboarding Page 생성
```
Onboarding Knowledge Pack 자동 생성은 신규 인원이 특정 도메인에 투입될 때 필요한 핵심 자료를 자동으로 묶어 제공하는 Workflow이다.
도메인 태그를 기준으로 5.2 Code Analyzer의 Call Flow MSC + 모듈소개, RCA Knowledge Graph, HLD 문서, TC, FAQ, Best Practice를 조회하여 Onboarding Page를 생성한다.
**Expected Benefit**
- 온보딩 기간 단축; 구두 전달 의존도 감소; 도메인 지식 재사용; 신규 인원 초기 분석 시간 감소; 과거 RCA 사례 전달; 필수 API와 HLD 누락 방지
**Precondition**
- 5.2 Code Analyzer (Track A) 완성 필요; 5.5 RCA Knowledge Graph 완성 필요; Team Knowledge 저장 구조 필요; 도메인 태그 기준 필요
**Storage / Reuse**
- Domain Guide; API List; HLD Link; RCA Case; TC Link; FAQ; Best Practice; Owner; 신규 인원 투입 시 자동 Page 생성; 반복 질문을 FAQ로 전환
---
## 6. Common Infrastructure
Common Infrastructure는 여러 자동화 Workflow에서 공통으로 사용하는 실행 모듈과 지식 기반을 제공한다.
Analyze와 Decision이 완료된 이후 실제 업무 시스템에 반영하기 위한 Action Module과, 자동화 아이템 전반이 공유하는 Knowledge System으로 구성된다.
### 6.1 Mail Notification Module
```text
Event
  ↓
AI 분석
  ↓
수신자 결정
  ↓
메일 초안 생성
  ↓
(선택) 메일 송신
```
Mail Notification Module은 자동화 Workflow의 결과를 필요한 담당자에게 알리는 역할을 한다.
Critical Defect 발견, Jira Close, HLD 업데이트, 주간보고 생성 등 Workflow 결과를 메일로 송신한다.
메일 초안에는 주요 내용, 필요한 액션, 관련 링크를 포함하여 수신자가 즉시 다음 단계로 진행할 수 있도록 한다.
**Expected Benefit**
- 팀 소통 개선; 알림 지연 감소; 중요 이슈 누락 방지; 액션 아이템 추적
**Precondition**
- 메일 서버 연동 필요; 수신자 리스트 기준 필요; 메일 템플릿 정의 필요
**Storage / Reuse**
- Mail Template; Recipient List; Mail History; Feedback
---
### 6.2 Confluence Integration Module
```text
Decision
  ↓
Confluence Page 생성/업데이트
  ↓
Link 생성
  ↓
액세스 권한 관리
```
Confluence Integration Module은 AI 자동화의 결과를 Confluence에 저장하고 관리하는 역할을 한다.
Code Analyzer의 MSC, HLD, Weekly Report 등의 산출물을 Confluence에 자동으로 업로드하고, 필요한 링크를 생성한다.
Jira와의 양방향 연동으로 Issue와 Confluence Page를 연결한다.
**Expected Benefit**
- 문서 중앙 집중식 관리; 팀 지식 축적; 협업 효율 개선; 온보딩 자료 자동 갱신
**Precondition**
- Confluence MCP 연동 필요; Page 구조 표준화 필요; 액세스 권한 정책 필요
**Storage / Reuse**
- Confluence Page; Page Link; Permission; Comment History; Page Version; 다음 주기 Page 생성 시 template 재사용
---
### 6.3 Jira Integration Module
```text
Defect / Issue
  ↓
Jira Ticket 생성
  ↓
상태 추적
  ↓
Close 또는 Reopen
  ↓
RCA / Prevent Rule 저장
```
Jira Integration Module은 AI 자동화의 판단 결과를 Jira로 연동하고, Jira 상태 변화를 추적하여 Feedback Loop를 완성하는 역할을 한다.
Critical Defect 검출 결과를 Jira Ticket으로 생성하고, Review 결과에 따라 Rule을 업데이트한다.
**Expected Benefit**
- Defect 관리 자동화; 추적 효율 개선; RCA와 Jira 연결; Prevent Rule 자동 생성
**Precondition**
- Jira MCP 연동 필요; Issue Type 정의 필요; Custom Field 정의 필요 (Shelve Number, Root Cause, Patch Proposal 등)
**Storage / Reuse**
- Jira Ticket; Comment; Link; Resolution; RCA Document; Rule DB; 다음 유사 Issue 검출 시 과거 RCA 참조
---
## 7. Implementation Priority
### Phase 1 (즉시 착수)
- 5.3-pre Confluence Child Page Collection (매주 반복; 5.3의 선행)
- 5.3 Weekly Report Collection (5.3-pre draft 확보 후; 반복 업무 즉시 효과)
- 5.2 Code Analyzer (독립 착수; 이후 5.4/5.5/5.6의 기반)
- 5.1 Jira Feedback Loop (독립 착수; Defect 관리 연결)
### Phase 2 (5.2 완료 후)
- 5.5 RCA Knowledge Graph (5.2 필수)
- 5.4 HLD ↔ Code Consistency Check (5.2 권장)
### Phase 3 (5.2 + 5.5 완료 후)
- 5.6 Onboarding Knowledge Pack (5.2 + 5.5 필수)
### Common Infrastructure (병렬)
- 6.1 Mail Notification Module (언제든 착수)
- 6.2 Confluence Integration Module (언제든 착수)
- 6.3 Jira Integration Module (5.1과 병렬)
---
## 8. Success Metrics
- Code Analyzer: 대용량 폴더(30분 이상) 완료율 100%; MSC 정확도 95% 이상
- Weekly Report: 매주 생성 시간 < 10분; Risk 추적 누락률 0%
- Jira Feedback: Review → Rule Update 순환 시간 < 2일; False Positive 감소율 20% 이상
- RCA Knowledge Graph: 유사 Issue 검색 정확도 80% 이상; 분석 시간 단축 30% 이상
- 종합: 팀원 반복 판단 업무 자동화율 > 70%; 월 총 절감 시간 > 40시간
---
## Version History
- **v0.36** (2026-01-XX): Initial framework design
- **v0.37** (2026-02-XX): 5.0 Common Framework section added
- **v0.38** (2026-03-XX): 5.3-pre Confluence Child Page Collection added
- **v0.39** (2026-04-XX): 5.3 Weekly Report finalization
- **v0.40** (2026-06-19): 5.2 Code Analyzer redesign with staged-code-analyzer, Track A/B, Global CNF Carry; Master 5.0 sections added; 5.4/5.5/5.6 updates
