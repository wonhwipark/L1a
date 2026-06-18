# L1 AI Automation Roadmap v0.39
---
## Document Purpose
이 문서는 L1 AI Automation Roadmap의 v0.39 Master Document이다.
이 문서 하나만으로 새 창에서도 v0.40, v1.0으로 계속 버전업할 수 있도록 독립형 문서로 구성한다.
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
Review
  ↓
Jira Close
  ↓
Rule DB Update
  ↓
Knowledge Layer 축적
```
현재 Critical Defect Workflow는 특정 Branch의 특정 Folder를 대상으로 Critical Defect 가능성을 검출하고, 수정안을 제안한 뒤, Patch를 생성하고, P4 Shelve를 만든 후 Jira 등록까지 연결하는 구조이다.
이 Workflow는 단순 코드 리뷰 보조가 아니라 실제 Action까지 이어지는 자동화 구조라는 점에서 중요하다.
현재 단계에서 이미 확보된 의미는 다음과 같다.
- AI가 코드 분석만 수행하는 것이 아니라 수정 후보를 제안한다.; 수정 후보를 실제 Patch 형태로 변환한다.; Patch를 P4 Shelve로 연결한다.; 검출 결과를 Jira 등록으로 연결한다.; 이후 Review와 Close 결과를 다시 Rule에 반영할 수 있는 기반을 가진다.
Critical Defect Workflow는 향후 모든 L1 자동화 Workflow의 Reference Workflow로 사용할 수 있다.
그 이유는 Input, Analyze, Decision, Action, Verify, Feedback, Knowledge Layer, Reuse 구조로 확장 가능하기 때문이다.
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
```text
코드 루트 경로 + 분석 대상 지정
  ↓
코드 구조 분석 (함수/클래스/호출 관계/Branch 조건)
  ↓
Call Flow 추출
  ↓
HLD 업데이트 항목 목록 생성
  ↓
HLD 문서 초안 생성
  ↓
(선택) Confluence 업로드 또는 로컬 저장
```
Code Analyzer는 HLD 없이 코드만 존재하는 기존 구현을 분석하여 HLD(또는 구현문서)를 문서화하는 Workflow이다.
개별 담당자가 자신의 담당 코드를 대상으로 직접 실행하며, 분석 결과로 HLD 업데이트 항목 목록과 HLD 문서 초안을 생성한다.
5.4 HLD ↔ Code Consistency Check와의 역할 구분: 5.2는 HLD가 없는 상태에서 코드로부터 HLD를 새로 만들고, 5.4는 HLD가 이미 있는 상태에서 코드와의 불일치를 찾는다.
**Expected Benefit**
- HLD 미작성 코드의 문서화 부채 해소; 신규 인원 온보딩 지원; 코드 리뷰 품질 향상; HLD 작성 시간 단축; Call Flow 가시화; 구현 의도 복원
**Precondition**
- 없음; 독립 착수 가능; 분석 대상 코드 경로 및 언어/확장자 지정 필요; HLD 출력 템플릿 정의 필요
**Storage / Reuse**
- HLD 업데이트 항목 목록; HLD 문서 초안; Call Flow MSC(PlantUML); 함수/클래스 구조 분석 결과 JSON; 5.4 Consistency Check 입력으로 재사용 가능; 5.5 RCA Knowledge Graph의 API 연결 정보로 재사용 가능; 5.6 Onboarding Knowledge Pack의 HLD 링크로 활용 가능
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
- 취합 원본 draft(`%USERPROFILE%\artifacts\weekly_report_draft_<YYYYMMDD>_<HHMM>_KST.md`); 선택적 strategy profile(`%USERPROFILE%\artifacts\child_page_discovery_profile.json`); 5.3 Weekly Report Collection의 입력으로 사용
---
### 5.3 Weekly Report Collection
```text
5.3-pre weekly_report_draft 입력
  ↓
Previous Week Page Confluence Link 기준 이전 주 양식 참조
  ↓
취합 원본 draft를 이전 주 양식에 맞춰 정리
  ↓
이번 주 TL 주간보고 final 작성
  ↓
Group Report
```
Weekly Report Collection은 5.3-pre 단계에서 생성한 Child Page 본문 취합 원본 draft를 입력으로 받고, Previous Week Page Confluence Link를 통해 이전 주 주간보고 양식을 참조하여 이번 주 TL 주간보고 final을 작성하는 Workflow이다.
Child Page 탐색과 본문 취합은 pre 단계에서 수행하며, 5.3은 draft 내용을 이전 주 양식에 맞춰 정리하고 이번 주 보고서를 생성한다.
이 과정은 반복성이 높고 누락 가능성이 있으므로 자동화 가치가 크다.
팀원 목록은 사용하지 않으며, 실제 발견된 Child Page 내용만 사용한다.
**Expected Benefit**
- 반복 업무 감소; 이전 주 양식 기반 일관성 유지; 주간보고 정리 시간 단축; 주차별 History 축적; TL 보고 final 작성 시간 감소
**Precondition**
- Confluence read 가능; 5.3-pre Child Page Collection draft 선행 생성 필요(`%USERPROFILE%\artifacts\weekly_report_draft_<YYYYMMDD>_<HHMM>_KST.md`); Previous Week Page Confluence Link 필요; Group Report 형식 정의 필요
**Storage / Reuse**
- Weekly Report final(`%USERPROFILE%\artifacts\weekly_report_final_<YYYYMMDD>_<HHMM>_KST.md`); 입력 draft(`%USERPROFILE%\artifacts\weekly_report_draft_<YYYYMMDD>_<HHMM>_KST.md`); Risk Trend; Action Item; History; 작성자; 주차; Confluence Link; 다음 주 Risk Trend 비교; 지연 Action Item 추적; 반복 Risk 검출
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
기능 변경 후 Code는 업데이트되었지만 HLD가 갱신되지 않거나, HLD에는 설계가 존재하지만 실제 구현에서 누락되는 경우를 검출한다.
**Expected Benefit**
- 설계 최신화; 문서 품질 향상; Code와 HLD 간 불일치 감소; Review 품질 향상; 신규 인원 이해도 향상; 기능 변경 이력 추적 강화
**Precondition**
- HLD 형식 표준화 필요; 5.2 Code Analyzer 구축 권장; Confluence Update 권한 필요; Code 분석 범위 지정 필요
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
- 5.2 Code Analyzer 구축 필요; Jira와 RCA 문서 연결 필요; Log Pattern 저장 기준 필요; Root Cause 분류 기준 필요
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
HLD 문서 연결
  ↓
Onboarding Page 생성
```
Onboarding Knowledge Pack 자동 생성은 신규 인원이 특정 도메인에 투입될 때 필요한 핵심 자료를 자동으로 묶어 제공하는 Workflow이다.
도메인 태그를 기준으로 API Call Flow DB, RCA Knowledge Graph, HLD 문서, TC, FAQ, Best Practice를 조회하여 Onboarding Page를 생성한다.
**Expected Benefit**
- 온보딩 기간 단축; 구두 전달 의존도 감소; 도메인 지식 재사용; 신규 인원 초기 분석 시간 감소; 과거 RCA 사례 전달; 필수 API와 HLD 누락 방지
**Precondition**
- 5.2 Code Analyzer 완성 필요; 5.5 RCA Knowledge Graph 완성 필요; Team Knowledge 저장 구조 필요; 도메인 태그 기준 필요
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
검토(Optional)
  ↓
메일 발송
  ↓
History 저장
```
Mail Notification Module은 특정 Event 발생 시 AI 분석 결과를 바탕으로 수신자를 결정하고 메일 초안을 생성한 뒤, 필요 시 사람 검토를 거쳐 메일을 발송하고 History를 저장하는 모듈이다.
메일 발송은 Critical Defect 알림, Jira Review 요청, Weekly Report Action Item 지연 알림, HLD와 Code 불일치 알림, RCA 공유, Onboarding Page 안내에 활용할 수 있다.
Mail Notification Module은 Human-in-the-loop 구조를 지원해야 한다.
### 6.2 Future Action Modules
**Jira**
- Create; Update; Close; Comment; Link; Status Check; Assignee Update
**Confluence**
- Create; Update; Read; Child Page Scan; Link Update; Table Update; Report Publish
**Perforce**
- Shelve; Submit; Diff; Branch Scan; File History 조회; Change List 조회; Review Patch 생성
**Teams**
- Notification; Review Request; Risk Alert; Daily Summary; Action Item Reminder
**Calendar**
- Meeting 생성; Review Meeting 생성; RCA Meeting 생성; Follow-up 일정 생성
**Dashboard**
- Risk Dashboard Update; Jira Status Dashboard Update; Defect Trend Update; Weekly Report Trend Update; Action Item Status Update
**Knowledge Layer**
- Rule 축적; API Call Flow 저장; RCA Knowledge Graph 업데이트; Prompt / Skill DB Record 저장; Weekly Report History 저장; Team Knowledge 업데이트
### 6.3 Action Module 설계 원칙
- Action은 가능한 한 구조화된 Input을 받아야 한다.; Action 수행 전 Preview를 제공할 수 있어야 한다.; Risk가 높은 Action은 Human Review를 거쳐야 한다.; Action 수행 결과는 Verify 단계에서 확인되어야 한다.; Action 수행 History는 저장되어야 한다.; 실패 시 Failure Scenario와 Recovery Action을 제공해야 한다.; 동일 Action은 여러 Workflow에서 재사용할 수 있어야 한다.
### 6.4 Common Knowledge System
Common Knowledge System은 5.1 Jira Feedback Loop, 5.2 Code Analyzer, 5.4 HLD↔Code Consistency Check, 5.5 RCA Knowledge Graph, 5.6 Onboarding Knowledge Pack이 공유하는 지식 축적 및 검색 기반이다.
Resolve/Reject 사례, Prevent Rule, False Positive Rule을 저장하고, 다음 분석 시 관련 사례를 선별하여 AI context에 주입한다.
현재 구현 범위: Phase 1 (Markdown 기록 + Skill Update 중심 운영).
상세 구성 및 Phase 1~4 Roadmap은 `prompt/6_4_common_knowledge_system_spec.md`를 참조한다.
---
## 7. 자동화 아이템 정의 템플릿
### 목적
모든 자동화 아이템을 동일한 기준으로 정의하기 위한 공통 템플릿이다.
단순 아이디어 수준의 항목을 실제 구현 가능한 Workflow Specification으로 발전시키는 것을 목표로 한다.
새로운 자동화 아이템을 추가할 때는 반드시 이 템플릿을 기준으로 Background, Goal, Workflow, Input, Analyze, Decision, Action, Verify, Output, Storage, Reuse를 명확히 정의한다.
### 7.1 Common Template
- Background; Goal; Workflow; Input; Analyze; Decision; Action; Verify; Output; Storage; Reuse; Expected Benefit; Success Metric; Precondition; Failure Scenario; Future Extension
### 7.2 Background
자동화 아이템이 필요한 업무 배경을 설명한다. 현재 사람이 반복하는 업무, 반복 판단, 현재 방식의 문제, 자동화하지 않을 경우의 비효율을 정의한다.
### 7.3 Goal
자동화 아이템의 목표를 정의한다. 목표는 단순 분석이 아니라 실제 업무 결과로 연결되어야 한다.
### 7.4 Workflow
자동화의 전체 흐름을 단계별로 정의한다. 가능한 경우 Input, Analyze, Decision, Action, Verify, Feedback, Knowledge Layer, Reuse 구조를 따른다.
### 7.5 Input
Workflow가 처리할 대상이다. 입력 데이터 종류, 위치, 형식, 범위, 필수 입력, 선택 입력, 입력 검증 기준을 포함한다.
### 7.6 Analyze
AI 또는 Rule Engine이 수행할 분석 내용을 정의한다. 분석 대상, 분석 기준, 참조할 Knowledge Layer, 분석 결과 형식, 신뢰도 또는 근거를 포함한다.
### 7.7 Decision
분석 결과를 바탕으로 수행할 판단 기준을 정의한다. Action 필요 여부, 우선순위, Risk Level, 담당자, Human Review 필요 여부, 자동 처리 가능 여부를 포함한다.
### 7.8 Action
실제 업무 시스템에 수행할 동작을 정의한다. Jira, Confluence, Perforce, Mail, Teams, Dashboard, Knowledge Layer Update를 포함할 수 있다.
### 7.9 Verify
Action이 정상적으로 수행되었는지 확인하는 단계이다. 결과 확인 방법, 성공 기준, 실패 기준, 재시도 기준, 사람이 확인해야 하는 항목을 포함한다.
### 7.10 Output
Workflow 수행 결과물이다. Patch, P4 Shelve, Jira, Confluence Page, MSC, HLD 수정안, RCA Summary, Weekly Report Draft, Mail Draft, Dashboard Update, Knowledge Layer Record가 될 수 있다.
### 7.11 Storage
Workflow 결과를 어디에 저장할지 정의한다. 저장소 종류, 저장 데이터, 저장 형식, Key, Version, Owner, Update 시점을 포함한다.
### 7.12 Reuse
저장된 결과를 어떤 업무에 다시 사용할지 정의한다. Rule, Call Flow, RCA, Prompt, Skill, Weekly Report History, Onboarding 자료 재사용을 포함한다.
### 7.13 Expected Benefit
자동화를 통해 기대하는 효과이다. 업무 시간 감소, 누락 감소, 품질 향상, 재분석 시간 감소, 온보딩 시간 감소, Review 품질 향상, Defect 재발 방지를 포함한다.
### 7.14 Success Metric
자동화 성공 여부를 측정하는 기준이다. 분석 시간 감소율, False Positive 감소율, Jira 처리 시간 감소, HLD Gap 검출 건수, 주간보고 작성 시간 감소, Onboarding 기간 감소, 재사용 건수를 포함한다.
### 7.15 Precondition
자동화 착수 전에 필요한 조건이다. MCP 인증, 저장소 접근 권한, HLD 형식 표준화, Jira Project Key, Confluence Space Key, Branch 접근 권한, Rule DB 초기 구성을 포함한다.
### 7.16 Failure Scenario
자동화가 실패할 수 있는 경우를 정의한다. 입력 데이터 불완전, MCP 인증 실패, Jira 등록 실패, Confluence Update 실패, Code 분석 범위 과대, False Positive 과다, Verify 실패를 포함한다.
### 7.17 Future Extension
자동화 아이템의 향후 확장 방향이다. Dashboard, Mail Notification, Rule Engine, Knowledge Graph, Onboarding 자동 생성, TC 자동 생성, Jira Close 자동 분석 연동을 포함한다.
---
## 8. Knowledge Architecture
Knowledge Architecture는 L1 AI 자동화의 결과가 일회성 산출물로 끝나지 않고 조직 자산으로 축적되도록 하기 위한 구조이다.
Knowledge Architecture의 핵심은 Knowledge Layer이다.
Knowledge Layer는 LLM 자체가 아니다.
LLM은 분석, 요약, 생성, 추론을 수행하는 엔진이지만, 조직의 장기 지식은 구조화된 저장소에 축적되어야 한다.
Knowledge Layer는 Rule DB, API Call Flow DB, RCA Knowledge Graph, Prompt / Skill DB, Report DB, Team Knowledge를 포함하는 구조화된 지식 계층이다.
핵심 개념은 다음과 같다.
```text
Human Experience
 ↓
AI Workflow
 ↓
Knowledge Layer
 ↓
Reuse
 ↓
Organizational Asset
```
사람의 경험이 개인의 기억이나 구두 전달에 머물면 조직 자산이 되기 어렵다.
AI Workflow는 사람의 경험을 구조화하여 Knowledge Layer에 저장하고, 저장된 지식을 다시 업무에 재사용하도록 만드는 역할을 한다.
### 8.1 Rule DB
Rule DB는 Critical Defect, False Positive, Prevent Rule, Review Feedback, Rule Version을 저장하는 구조화된 지식 저장소이다.
Rule DB는 LLM 자체가 아니라 구조화된 지식 저장소이다.
LLM은 Rule DB를 조회하여 분석 기준을 이해하고, 신규 Rule 후보를 제안하고, Review 결과를 요약할 수 있다.
하지만 Rule의 원본과 버전, 적용 조건, 예외 조건은 Rule DB에 저장되어야 한다.
**저장 대상**
- Critical Defect Rule; False Positive History; Prevent Rule; Review Feedback; Rule Version
Critical Defect Rule은 코드 변경에서 반드시 검출해야 하는 위험 패턴이다.
예시는 Array index out-of-bound 가능성, Set 함수와 Get 함수의 size mismatch, Domain index mismatch, RAT별 분기 누락, Dual SIM 조건 누락, ENDC 조건 누락, ULCA 조합 조건 누락, Timer rollover 미고려, Invalid value 처리 누락이다.
False Positive History는 AI 또는 Rule Engine이 Defect로 판단했지만 Review 결과 실제 Defect가 아니었던 사례를 저장한다.
Prevent Rule은 RCA 또는 Review 결과를 바탕으로 재발 방지를 위해 생성하는 Rule이다.
Review Feedback은 Reviewer가 남긴 판단 근거, 수정 요청, 예외 인정, Rule 수정 의견을 저장한다.
Rule Version은 Rule 변경 이력을 관리하기 위한 정보이며 Rule ID, Rule Name, Rule Description, Severity, 적용 조건, 예외 조건, 생성 근거, 생성 일자, 수정 일자, Owner, 적용 Workflow를 포함할 수 있다.
**구현 형태**
- JSON; YAML; SQLite; Vector DB
JSON은 단순 Rule 저장과 교환에 적합하다.
YAML은 사람이 읽고 수정하기 쉬운 Rule 정의에 적합하다.
SQLite는 Local 환경에서 Rule History와 Version을 관리하기에 적합하다.
Vector DB는 유사 Defect, 유사 Review Feedback, 유사 False Positive 검색에 적합하다.
**Rule Engine**
- Drools; OpenL Tablets
Drools는 복잡한 조건 기반 Rule 실행에 사용할 수 있다.
OpenL Tablets는 테이블 기반 Rule 관리가 필요한 경우 사용할 수 있다.
Rule Engine은 LLM과 별개로 결정 가능한 Rule을 안정적으로 실행하기 위한 구성 요소이다.
### 8.2 API Call Flow DB
API Call Flow DB는 API를 시작점으로 Call Flow, MSC, Scenario, Known Defect, TC를 연결하여 저장하는 구조화된 지식 저장소이다.
구조
```text
API
 ↓
Call Flow
 ↓
MSC
 ↓
Scenario
 ↓
Known Defect
 ↓
TC
```
API Call Flow DB는 L1 코드 분석 결과를 재사용 가능한 형태로 저장하기 위한 핵심 저장소이다.
특정 API 분석 결과가 문서에만 남으면 다음 분석 시 다시 코드를 추적해야 한다.
하지만 API Call Flow DB에 구조화해두면 API 영향도 분석, HLD 작성, MSC 작성, RCA 분석, TC 설계에 재사용할 수 있다.
**저장 대상**
- API; Caller; Callee; Branch Condition; Message; Function; IPC; HAL Interface; RF DRV Interface; PHY Interface; MSC; Scenario; Known Defect; TC
API는 분석의 시작점이며 함수명, 클래스명, 파일 경로, RAT, Domain, Feature Tag를 포함할 수 있다.
Call Flow는 API가 호출된 이후 실제 코드가 어떤 경로로 실행되는지 나타내며 Normal Flow와 Exception Flow를 모두 포함해야 한다.
MSC는 Call Flow를 시퀀스 형태로 표현한 산출물이며 HLD 작성과 Review에 유용하다.
Scenario는 Call Flow가 발생하는 조건과 사용 사례를 설명하며 TC 설계와 RCA 분석의 연결점이다.
Known Defect는 해당 API 또는 Scenario에서 과거 발생한 Defect를 연결한다.
TC는 Scenario를 검증하기 위한 Test Case이며 Code 변경 시 영향 받는 TC를 추적할 수 있게 한다.
### 8.3 RCA Knowledge Graph
RCA Knowledge Graph는 Issue, Log, Root Cause, Fix, Jira, HLD, TC, API를 연결하는 구조화된 지식 그래프이다.
RCA Knowledge Graph 역시 LLM 자체가 아니라 구조화된 지식 저장소이다.
LLM은 RCA 문서를 요약하고 관계 후보를 제안할 수 있지만, 관계의 저장과 재사용은 RCA Knowledge Graph에서 수행해야 한다.
구조
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
```
**구현 형태**
- JSON; Neo4j; RDF Graph; Knowledge Graph
JSON은 초기 Proof of Concept에 적합하다.
Neo4j는 Node와 Edge 기반의 관계 검색에 적합하다.
RDF Graph는 표준 Ontology와 Semantic Query가 필요한 경우 적합하다.
Knowledge Graph는 RCA 재사용과 유사 Issue 검색을 위한 장기 구조로 적합하다.
**Node 예시**
- Issue; Log; Root Cause; Fix; Jira; HLD; TC; API; Owner; Module; Feature
**Edge 예시**
- Issue has Log; Log indicates Root Cause; Root Cause fixed by Fix; Fix linked to Jira; Jira updates HLD; HLD requires TC; TC covers API; API related to Issue
**활용 예시**
- 특정 Log Pattern으로 과거 Issue 검색; 특정 API와 연결된 Known Defect 검색; 특정 Root Cause 유형의 재발 여부 확인; 특정 Fix가 반영된 HLD와 TC 확인; 신규 Issue 발생 시 유사 RCA 후보 제공
### 8.4 Prompt / Skill DB
Prompt / Skill DB는 Roo Skill, Claude Skill, Prompt History, Workflow History, Success Pattern을 저장하는 구조화된 지식 저장소이다.
Prompt와 Skill은 AI Workflow 품질에 직접 영향을 준다.
그러나 Prompt와 Skill이 개인별 파일이나 대화 이력에 흩어져 있으면 재사용과 개선이 어렵다.
Prompt / Skill DB는 자동화 Workflow에서 검증된 Prompt와 Skill을 저장하고 버전업하기 위한 저장소이다.
**저장 대상**
- Roo Skill; Claude Skill; Prompt History; Workflow History; Success Pattern
Roo Skill은 Roo Code 환경에서 반복 업무를 수행하기 위한 규칙과 절차이다.
Claude Skill은 Claude Code 환경에서 반복 업무를 수행하기 위한 규칙과 절차이다.
Prompt History는 실제 업무에서 사용한 Prompt와 결과를 저장하며 성공 사례와 실패 사례를 모두 포함해야 한다.
Workflow History는 자동화 Workflow가 어떤 입력으로 실행되었고 어떤 결과를 냈는지 저장한다.
Success Pattern은 성공적으로 동작한 Prompt, Skill, Workflow 조합을 저장한다.
### 8.5 Report DB
Report DB는 Weekly Report, Risk Trend, Action Item, History를 저장하는 구조화된 지식 저장소이다.
Report DB는 주간보고 자동화와 Risk 추적 자동화를 위한 기반이다.
주간보고는 단순 문서가 아니라 조직의 상태, 위험, 지연, 의사결정 필요 항목을 담고 있다.
**저장 대상**
- Weekly Report; Risk Trend; Action Item; History
Weekly Report는 개인별 주간보고와 TL 주간보고, 그룹 보고를 포함한다.
Weekly Report에는 작성자, 주차, 도메인, 주요 업무, Risk, Action Item, 완료 항목, 지연 항목을 포함할 수 있다.
Risk Trend는 특정 Risk가 시간에 따라 증가하는지 감소하는지 추적한다.
Action Item은 담당자, 기한, 상태, 후속 조치가 필요한 항목이다.
History는 주차별 보고 변화, 반복 Risk, 지연 Action Item, Close 이력을 저장한다.
### 8.6 Team Knowledge
Team Knowledge는 Domain Guide, Onboarding Package, FAQ, Best Practice를 저장하는 조직 지식 저장소이다.
Team Knowledge는 개인의 경험을 팀 전체가 재사용할 수 있는 형태로 전환하기 위한 Knowledge Layer의 일부이다.
**저장 대상**
- Domain Guide; Onboarding Package; FAQ; Best Practice
Domain Guide는 도메인별 핵심 구조, 주요 API, 주요 HLD, 주요 TC, 담당자, 주의사항을 정리한 문서이다.
Onboarding Package는 신규 인원이 특정 도메인에 투입될 때 필요한 자료 묶음이다.
Onboarding Package에는 API Call Flow DB, RCA Knowledge Graph, HLD, TC, FAQ, Best Practice가 연결될 수 있다.
FAQ는 반복 질문과 답변을 구조화한 지식이다.
Best Practice는 검증된 설계 방식, 코드 작성 방식, Review 기준, Debugging 방식, Report 작성 방식을 저장한다.
### 8.7 Knowledge Layer 운영 원칙
- LLM 결과를 그대로 장기 지식으로 저장하지 않는다.; 구조화된 데이터 형태로 저장한다.; Version을 관리한다.; Owner를 지정한다.; 검증된 정보와 후보 정보를 구분한다.; 사람의 Review 결과를 반영한다.; Workflow 실행 결과와 연결한다.; Reuse 가능한 형태로 저장한다.
Knowledge Layer의 품질은 자동화 품질과 직접 연결된다.
초기에는 Human Review를 포함한 운영이 필요하다.
### 8.8 Knowledge Architecture 통합 구조
```text
Rule DB
  ↓
Critical Defect Workflow
API Call Flow DB
  ↓
HLD / MSC / TC / Onboarding
RCA Knowledge Graph
  ↓
Issue Analysis / Prevent Rule / Known Defect
Prompt / Skill DB
  ↓
Workflow Quality Improvement
Report DB
  ↓
Weekly Report / Risk Trend / Action Item Tracking
Team Knowledge
  ↓
Domain Guide / Onboarding / FAQ / Best Practice
```
각 DB는 독립적으로 존재하지만 서로 연결되어야 한다.
RCA Knowledge Graph에서 특정 API가 연결되면 해당 API는 API Call Flow DB의 Call Flow와 연결되어야 한다.
Critical Defect Rule은 RCA Knowledge Graph의 Root Cause와 연결되어야 한다.
Weekly Report의 Risk는 Jira, RCA, Action Item과 연결될 수 있어야 한다.
Prompt / Skill DB는 각 Workflow의 성공과 실패 결과를 반영해야 한다.
---
## 9. Long-term Vision
L1 AI Automation의 장기 목표는 개별 업무 보조가 아니라 L1 Engineering Operation 자체를 자동화하는 것이다.
```text
Detect
 ↓
Analyze
 ↓
Decision
 ↓
Action Module
 ↓
Verify
 ↓
Feedback
 ↓
Knowledge Layer
 ↓
Reuse
```
Detect는 Issue, Defect, Risk, Gap, Action Item을 발견하는 단계이다.
Analyze는 발견된 항목을 코드, 로그, HLD, Jira, Confluence, RCA, TC와 연결하여 해석하는 단계이다.
Decision은 분석 결과를 바탕으로 Action 필요 여부와 우선순위를 판단하는 단계이다.
Action Module은 Jira, Confluence, Perforce, Mail, Teams, Calendar, Dashboard, Knowledge Layer에 실제 동작을 수행하는 단계이다.
Verify는 수행된 Action이 의도대로 완료되었는지 확인하는 단계이다.
Feedback은 Verify 결과와 사람의 Review 결과를 Rule, Prompt, Skill, Workflow에 반영하는 단계이다.
Knowledge Layer는 모든 결과를 구조화된 지식으로 저장하는 단계이다.
Reuse는 저장된 지식을 다음 업무에 다시 사용하는 단계이다.
최종 목표
```text
AI-driven L1 Engineering Operation Platform
↓
Autonomous L1 Engineering Platform
```
AI-driven L1 Engineering Operation Platform은 사람이 주도하고 AI가 분석, 판단 보조, Action 초안 생성, Verify 보조를 수행하는 단계이다.
Autonomous L1 Engineering Platform은 반복적인 L1 운영 업무가 Closed-loop로 자동 수행되고, 사람은 예외 상황과 최종 의사결정에 집중하는 단계이다.
### 9.1 Phase 1: Workflow 자동화
대상은 Critical Defect Workflow, API Call Flow 분석, Weekly Report Collection, HLD ↔ Code Consistency Check, Jira Feedback Loop, RCA 정리이다. 이 단계에서는 Human Review가 필수이다.
### 9.2 Phase 2: Action Module 통합
대상은 Jira Create, Jira Update, Confluence Create, Confluence Update, P4 Shelve, Mail Notification, Teams Notification, Dashboard Update이다. 이 단계에서는 Verify 구조가 중요하다.
### 9.3 Phase 3: Knowledge Layer 축적
대상은 Rule DB, API Call Flow DB, RCA Knowledge Graph, Prompt / Skill DB, Report DB, Team Knowledge이다. 이 단계부터 자동화는 일회성 도구가 아니라 조직 자산으로 발전한다.
### 9.4 Phase 4: Feedback 기반 품질 개선
대상은 Rule 개선, Prompt 개선, Skill 개선, Workflow 개선, False Positive 감소, Prevent Rule 강화이다. 이 단계에서는 자동화가 실행될수록 품질이 개선된다.
### 9.5 Phase 5: Autonomous Operation
대상은 반복 Defect 검출 자동화, 반복 Report 취합 자동화, 반복 HLD Gap 검출 자동화, 반복 Jira Feedback 분석 자동화, 반복 RCA 재사용 자동화, 반복 Onboarding Package 생성 자동화이다. 사람은 최종 의사결정, 예외 처리, 방향 설정, Rule 승인, 품질 검증을 담당한다.
---
## 10. Version-up Guideline
본 문서는 v0.39 Master Document이다.
버전업 운영 절차는 `readme_workflow.md`를 따른다.
향후 v0.40, v1.0으로 발전할 때는 다음 기준을 따른다.
- 기존 장의 핵심 내용을 삭제하지 않는다.; 용어는 Knowledge Layer 기준으로 유지한다.; RCA 관련 지식 구조는 RCA Knowledge Graph로 표기한다.; API 흐름 저장소는 API Call Flow DB로 표기한다.; Prompt와 Skill 이력 저장소는 Prompt / Skill DB로 표기한다.; 자동화 아이템 추가 시 7장 템플릿을 사용한다.; 신규 저장소 또는 DB 추가 시 8장 Knowledge Architecture와 연결한다.; 신규 Action 추가 시 6장 Common Infrastructure와 연결한다.; 장기 목표는 9장 Long-term Vision과 정합성을 유지한다.
v0.40에서는 실제 구현 우선순위, MVP 범위, 도구 연동 방식, 데이터 저장 구조를 구체화할 수 있다.
v1.0에서는 운영 가능한 L1 AI Automation 기준 문서로 확정할 수 있다.
---
## 11. Summary
- 단순 AI 활용이 아니라 Closed-loop 기반 업무 자동화를 목표로 한다.; 자동화 대상은 사람이 반복적으로 판단하는 업무를 기준으로 선정한다.; Existing Automation인 Critical Defect Workflow를 Reference Workflow로 삼는다.; Key Automation Items는 Jira Feedback Loop, API Call Flow DB, Weekly Report Collection, HLD ↔ Code Consistency Check, RCA Knowledge Graph, Onboarding Knowledge Pack 자동 생성이다.; Common Action Module은 Mail, Jira, Confluence, Perforce, Teams, Calendar, Dashboard, Knowledge Layer를 포함한다.; 모든 자동화 아이템은 공통 템플릿으로 정의한다.; Knowledge Architecture는 Rule DB, API Call Flow DB, RCA Knowledge Graph, Prompt / Skill DB, Report DB, Team Knowledge로 구성한다.; Long-term Vision은 AI-driven L1 Engineering Operation Platform에서 Autonomous L1 Engineering Platform으로 발전하는 것이다.
본 문서는 L1 업무 자동화의 Master Document이며, 이후 버전업의 기준 문서로 사용한다.
---
## v0.39 Change Summary

**Topic: topic04-ext3 — 5.3-pre 신규 추가 및 5.3.0 삭제**

### 변경 항목

1. **5.3-pre 신규 추가** — Confluence Child Page Collection
   - 5.3.0이 담당하던 Child Page 탐색 strategy 검증과 5.3이 담당하던 Child Page 본문 취합을 통합
   - 매주 실행되는 AI 실행 프롬프트 형태로 운용. 추후 Python 패키지로 구현 예정
   - 탐색 성공 시 선택적으로 strategy profile 저장 (apply 모드 시에만)
   - 출력: `weekly_report_draft_<YYYYMMDD>_<HHMM>_KST.md`

2. **5.3.0 삭제** — 5.3-pre에 기능 통합되어 별도 항목 불필요
   - `prompt/5_3_0_confluence_child_page_discovery_strategy_prompt.md` 삭제
   - Master 5.3.0 섹션을 5.3-pre 섹션으로 대체

3. **5.3 역할 범위 재정의** — Child Page 탐색 분리
   - 입력: 5.3-pre가 생성한 `weekly_report_draft_<YYYYMMDD>_<HHMM>_KST.md`
   - 출력: `weekly_report_final_<YYYYMMDD>_<HHMM>_KST.md`
   - Child Page 탐색 및 본문 취합은 더 이상 5.3에서 수행하지 않음

### 버전업 기준

- 5.3.0 항목 삭제 및 5.3-pre 신규 추가로 Master 본문 구조 변경 발생 → v0.39 승격

---

## v0.38 Change Summary

**Topic: topic04 — prompt folder naming + Confluence Child Page Discovery Strategy**

### 변경 항목

1. **5.3.0 신규 추가** — Confluence Child Page Discovery Strategy
   - 5.3 Weekly Report Collection의 선행 의존성 항목으로 신규 추가
   - Parent Page → Child Page 탐색 방식 검증 → Discovery Strategy Profile 저장
   - Profile 저장: 성공 method / confidence / fallback list 만 포함. 특정 주차 데이터 미포함
   - Profile 파일: `%USERPROFILE%\artifacts\child_page_discovery_profile.json` (고정, 재실행 시 overwrite)

2. **5.3 Workflow 갱신** — Child Page Scan 단계에 5.3.0 profile 활용 명시

3. **5.3 Precondition 갱신** — 5.3.0 profile 선행 생성 필요 조건 추가

4. **5.3 Expected Benefit 갱신** — "작성 누락 검출" 항목 제거 (팀원 목록 미사용 설계 반영)

5. **5.3 Storage/Reuse 갱신** — artifact 기준 경로(`%USERPROFILE%\artifacts`) 명시

6. **prompt/ 폴더명 확정** (`automation_prompt → skills → prompt` 최종 확정)
   - 5.x 파일은 runtime skill이 아니라 Python 구현 지시서이므로 `prompt/` 채택

7. **5.0.8 Artifact Management 갱신** — `ARTIFACTS_BASE_DIR = %USERPROFILE%\artifacts` 기준 경로 단일 정의

### 버전업 기준

- 5.3.0 신규 항목 추가로 Master 본문 구조 변경 발생 → v0.38 승격

---

## v0.37 Change Summary

v0.37에서는 5.1 Jira Feedback Loop 흐름 재정의, 6장 명칭 변경 및 6.4 Common Knowledge System 추가, automation_specs 폴더명 변경을 반영하였다.

주요 변경사항은 다음과 같다.

- 5.1 Jira Feedback Loop: 기존 단선 흐름을 7단계 번호 기반으로 재정의하였다. AI Root Cause Analysis, AI Patch Proposal, P4 Shelve Creation(AI 직접 생성)을 명시 단계로 추가하였다. Jira Close 이후 Reject/Resolve 분기 구조를 도입하였으며, Jira MCP 폴링(JQL)으로 Close 상태를 감지하고 Resolution 필드로 분류한다.
- 6장 명칭 변경: `Common Action Module` → `Common Infrastructure`로 변경하였다. Action Module과 Knowledge System을 모두 포괄하는 명칭으로 확장하였다.
- 6.4 Common Knowledge System 신규 추가: 5.1~5.6이 공유하는 지식 축적 및 검색 기반을 6장 하위로 위치를 확정하였다. 현재 구현 범위는 Phase 1(Markdown + Skill Update)이다. 상세 설계는 `prompt/6_4_common_knowledge_system_spec.md`를 참조한다.
- `automation_specs/` → `prompt/` 폴더명 변경: 구현 전용 프롬프트 저장소임을 명칭에 반영하였다.

---
## v0.35 Change Summary

v0.35에서는 v0.34의 Section 0(Document Update Operation Guide)을 제거하고, 버전업 운영 절차를 별도 파일로 분리하였다.

주요 변경사항은 다음과 같다.

- Section 0 전체 제거: 버전업 운영 절차 상세 내용이 Roadmap 본문과 중복되어 비대화를 유발하였다.
- `readme_workflow.md` 신설: GPT ↔ Claude 협업 기반 Delta Workflow 운영 절차를 별도 파일로 분리하였다. 버전업 운영에 관한 모든 상세 절차는 이 파일을 참조한다.
- Document Purpose 간소화: v0.34의 운영 방식 언급을 제거하고 `readme_workflow.md` 참조 한 줄로 대체하였다.
- Version-up Guideline 갱신: v0.35 기준으로 업데이트하고, 운영 절차 참조를 `readme_workflow.md`로 명시하였다.

---
## Appendix A. Terminology
### Knowledge Layer
L1 AI 자동화 결과가 저장되는 구조화된 지식 계층이다. Rule DB, API Call Flow DB, RCA Knowledge Graph, Prompt / Skill DB, Report DB, Team Knowledge를 포함한다.
### Rule DB
Critical Defect Rule, False Positive History, Prevent Rule, Review Feedback, Rule Version을 저장하는 구조화된 지식 저장소이다.
### API Call Flow DB
API, Call Flow, MSC, Scenario, Known Defect, TC를 연결하여 저장하는 구조화된 지식 저장소이다.
### RCA Knowledge Graph
Issue, Log, Root Cause, Fix, Jira, HLD, TC, API를 연결하는 구조화된 지식 그래프이다.
### Prompt / Skill DB
Roo Skill, Claude Skill, Prompt History, Workflow History, Success Pattern을 저장하는 구조화된 지식 저장소이다.
### Report DB
Weekly Report, Risk Trend, Action Item, History를 저장하는 구조화된 지식 저장소이다.
### Team Knowledge
Domain Guide, Onboarding Package, FAQ, Best Practice를 저장하는 조직 지식 저장소이다.
### Common Action Module
Jira, Confluence, Perforce, Mail, Teams, Calendar, Dashboard, Knowledge Layer에 실제 Action을 수행하는 공통 실행 모듈이다.
### Closed-loop
Input, Analyze, Decision, Action, Verify, Feedback, Knowledge Layer, Reuse로 이어지는 자동화 구조이다.
