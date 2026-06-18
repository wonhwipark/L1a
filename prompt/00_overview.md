# prompt Overview

이 폴더는 [`L1a/`](L1a) 저장소의 5장 Key Automation Items에 대한 항목별 Python 패키지 구현 프롬프트 저장소이다.

---

## 전제

`ai-mcp-package-v1.0_workflow_todo` 기반 환경 준비가 선행된 이후의 구현 전용 문서이다.
MCP 설치/설정 절차는 포함하지 않는다.

```text
- Roo Code / Claude Code 사용 가능
- Jira 생성 가능
- Confluence read 가능
- Confluence write/update 미확인 (각 패키지에서 TODO/로컬 출력으로 처리)
```

---


## 공통 설계 원칙

5.x 구현 프롬프트의 공통 원칙은 아래 단일 출처를 따른다.

```text
prompt/5_0_common_automation_framework.md
```

이 문서에서는 중복 서술 대신 아래만 유지한다.

- GPT + 사용자는 `What`과 업무 목적을 정한다.
- Roo Code / Claude Code는 환경 조사 후 `How`를 최적화한다.
- 구현 중 개선점은 [`delta/`](L1a/delta) 또는 [`review_logs/`](L1a/review_logs) 에 기록한다.

---


## 5.0 Common Automation Framework

상세 공통 Framework는 [`prompt/5_0_common_automation_framework.md`](L1a/prompt/5_0_common_automation_framework.md) 를 기준으로 한다.

개별 [`5_1`](L1a/prompt/5_1_jira_feedback_loop_py_package_prompt.md)~[`5_6`](L1a/prompt/5_6_onboarding_knowledge_pack_py_package_prompt.md) 은 이를 재사용한다.

---

## 항목 목록

| 파일 | 항목 | 핵심 목적 |
|------|------|----------|
| `5_0_common_automation_framework.md` | Common Automation Framework | 5.1~5.6 공통 철학, Layer, Artifact, Skill Loading 기준. ARTIFACTS_BASE_DIR 기준 경로 정의 포함. Staged Analysis (§5.0.10), Prefer Existing Environment (§5.0.5) 포함 |
| `5_1_jira_feedback_loop_py_package_prompt.md` | Jira Feedback Loop | Critical Defect → P4 Shelve → Jira 등록 → RCA → Prevent Rule |
| `5_2_code_analyzer_track_a_prompt.md` | Code Analyzer — Track A (권장, 메인 경로) | 대용량 폴더 단계별 분석 → Call Flow MSC (MSC-primary) + 간단 모듈소개 |
| `5_2_code_analyzer_track_b_prompt.md` | Code Analyzer — Track B (선택적, 사전 단계) | bash 정적 구조 추출 → JSON 메타데이터. Track A 진입 전 사전 단계 |
| `5_2_code_analyzer_py_package_prompt.md` | **[폐기 예정]** Code Analyzer — Legacy | HLD 없는 기존 코드 분석 → HLD 초안 생성 (v0.41부터 제거 계획) |
| `5_3_pre_confluence_child_page_collection_prompt.md` | Confluence Child Page Collection | Parent Page URL 기준 Child Page 탐색 → 본문 취합 → weekly_report_draft 생성. 추후 Python 패키지로 구현 예정 |
| `5_3_weekly_report_collection_py_package_prompt.md` | Weekly Report Collection | 5.3-pre draft 입력 → 이전 주 양식 기반 이번 주 final 작성 |
| `5_4_hld_code_consistency_check_py_package_prompt.md` | HLD ↔ Code Consistency Check | HLD 있는 상태에서 코드와 불일치 Gap 탐지 |
| `5_5_rca_knowledge_graph_py_package_prompt.md` | RCA Knowledge Graph | Issue/Log/RCA/Fix/Jira/HLD/TC/API 연결 지식 그래프 구축 |
| `5_6_onboarding_knowledge_pack_py_package_prompt.md` | Onboarding Knowledge Pack | 도메인 태그 기반 신규 인원 온보딩 자료 자동 생성 |

---

## 항목 간 의존 관계

```text
5.1    ────────────────────────────────────────────── 독립

5.2    ┌─ Track A (권장, 메인 경로)
       │  └─ staged-code-analyzer Skill 사용
       │
       └─ Track B (선택적, 사전 단계)
          └─ bash 정적 추출
       
       또는 Legacy py_package ([폐기 예정])
       ────────────────────────────────────────────── 독립

5.3-pre ────────────────────────────────────────────── 독립 (5.3의 선행 실행 필요)

5.3    ── (5.3-pre draft 필요) ─────────────────────── 의존

5.4    ── (5.2 Track A 또는 Legacy 완료 권장) ─────── 부분 의존
       (Track B 단독은 불충분 — Call Flow MSC 필수)

5.5    ── (5.2 Track A 또는 Legacy 완료 필수) ─────── 의존
       (Track B 단독은 불충분 — API 연결 정보 불완전)

5.6    ── (5.2 Track A 또는 Legacy + 5.5 완료 필수) ─ 강한 의존
```

---

## 권장 구현 순서 (우선순위 기준)

| Phase | 항목 | 이유 |
|-------|------|------|
| Phase 1 | **5.3-pre Confluence Child Page Collection** | 매주 실행. Child Page 탐색 → 본문 취합 → draft 생성. 5.3의 선행 조건. 추후 Python 패키지로 구현 예정 |
| Phase 1 | **5.3 Weekly Report Collection** | 5.3-pre draft 확보 후 착수. 이전 주 양식 기반 final 생성으로 반복 업무 즉시 효과 |
| Phase 1 | **5.2 Code Analyzer** | 독립 착수 가능, 이후 5.4·5.5·5.6의 기반 |
| Phase 1 | **5.1 Jira Feedback Loop** | 독립 착수 가능, Defect 관리 연결 |
| Phase 2 | **5.5 RCA Knowledge Graph** | 5.2 완료 후 착수, 5.6의 선행 조건 |
| Phase 2 | **5.4 HLD ↔ Code Consistency Check** | 5.2 완료 권장 후 착수 |
| Phase 3 | **5.6 Onboarding Knowledge Pack** | 5.2 + 5.5 완료 후 착수 |

---

## 5.2 Code Analyzer — Track 선택 가이드

### Track A 사용 (권장 — 메인 경로)

**조건:**
- 대용량 폴더 분석 (30분 이상 예상)
- Call Flow MSC 시각화 필요
- 신규 인원 빠른 이해 (간단 모듈소개)

**준비물:**
- Claude Code 또는 Roo Code
- staged-code-analyzer Skill 로드 필요 (07_skills_v1_3.zip)
- 분석 대상 폴더 + 단계 범위 사전 지정

**실행:**
- `5_2_code_analyzer_track_a_prompt.md`를 Claude Code/Roo Code에 입력
- §5 Section 9 "실행 프롬프트" 직접 사용

### Track B 사용 (선택적 — 사전 단계)

**조건:**
- Track A 진행 전 "전체 코드 구조 파악" 필요
- bash 환경에서 빠른 인벤토리 원할 때 (~5분)

**준비물:**
- bash 환경 + find/wc/ctags/grep 기본 도구
- 분석 대상 폴더

**실행:**
- `5_2_code_analyzer_track_b_prompt.md`를 Claude Code/Roo Code에 입력
- 생성된 JSON 메타데이터를 Track A의 "분석 대상 폴더 지정" 단계에서 활용 (선택적)

### Legacy py_package 사용 ([폐기 예정])

**상태:** v0.40부터 단계적 제거 예정. v0.41 이후 파일 삭제 계획.

**현재 사용 시 주의:**
- `5_2_code_analyzer_py_package_prompt.md`는 참고용만 사용
- 신규 분석은 Track A 또는 Track B 사용 권장
- 향후 더 이상 지원되지 않을 예정

### 권장 순서

1. **1차:** Track B로 구조 파악 (~5분)
2. **2차:** Track A 진입, staged-code-analyzer로 단계별 분석 (~20-30분, 폴더 크기에 따라)
3. **결과:** Call Flow MSC + 간단 모듈소개 + JSON 메타데이터

Track B 스킵 가능 (Track A 단독 사용 가능).

---

## 운영 원칙

- 각 파일은 독립 실행 단위 AI 구현 프롬프트이다.
- Section 9의 프롬프트를 Roo Code 또는 Claude Code에 직접 입력하여 구현을 시작한다.
- 선행 항목이 미완성인 경우, 각 파일 Section 9의 "기본 전제"에 stub 데이터 대체 방법이 명시되어 있다.
