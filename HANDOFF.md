# RCA Standalone — Handoff

이 패키지는 **L1a Master Roadmap의 5.5 RCA Knowledge Graph**를 L1a 본 워크플로(S0~S9)에서
분리하여 별도 트랙으로 진행하기 위해 추출한 것이다.

새 세션에서는 이 파일을 가장 먼저 읽는다.

---

## 1. 분리 배경

원래 5.5는 L1a Master Roadmap v0.40 §5.5에 속해 있었고, 확장안(`topic06` — 대용량 로그
Pre-filter Step 1)이 S4 Review까지 진행되었으나 사용자 판단으로 **Delta-only HOLD** 상태였다.

HOLD 상태에서 사용자가 사내에서 **L1SW Log Analyzer**라는 별도 스킬을 이미 전달받아 쓰고
있다는 사실이 확인되었고, 이 스킬의 로그 필터링/축소 기능이 5.5가 다루려는 문제(대용량
로그를 어떻게 줄여 분석할 것인가)와 겹칠 가능성이 제기되었다. 이를 깊이 검토하기 위해
RCA 관련 자산만 별도로 추출하여 새 세션/새 트랙으로 분리했다.

**즉, 이 패키지는 L1a S0~S7 워크플로(GPT delta ↔ Claude review ↔ Master 병합)의 적용
대상이 아니다.** RCA는 이제 독립적으로 검토·설계된다.

---

## 2. 이 패키지에 들어있는 것

```text
RCA_standalone/
├── HANDOFF.md                                  ← 이 파일
├── reference/
│   └── master_v0.40_section_5_5.md             ← L1a Master §5.5 원문 발췌 (참조용 사본)
├── prompt/
│   ├── 5_5_rca_knowledge_graph_py_package_prompt.md   ← 5.5 구현 프롬프트 (Step1/Step2 정의)
│   └── deepdive/                               ← ★ 5.5 별도 논의 자료
│       ├── L1SW_LOG_ANALYZER_COMPARISON.md      ← L1SW vs 5.5 비교 분석 (1차 세션)
│       ├── L1SW_VERIFICATION_QUESTIONS_v2.md    ← L1SW 확인 질문 15개 서술형 (2차 세션)
│       ├── L1SW_CHECKLIST_OBJECTIVE_v2.md       ← 같은 15개 객관식 (현장용, 2차 세션)
│       ├── session_log_20260621_KST.md          ← 2차 세션 논의 요약
│       ├── L1SW_AND_FLOW_20260620.md            ← ★ 온사이트 A~F 확인 결과, Option A 확정본
│       │                                          (L1SW 동작 정리 + L1SW→5.5 전체 Flow)
│       └── RCA_FEEDBACK_LOOP_DESIGN_20260622_KST.md  ← ★★ 3차 세션 — 피드백 루프 전체 설계
│                                                  (manifest 보강, fingerprint, unresolved/
│                                                   통합 워크플로, schema v2 결정 이력)
├── delta/
│   └── v0.40_topic06_gpt_delta_..._20260619_KST.md    ← topic06 GPT Delta (HOLD된 확장 제안)
├── review_logs/
│   ├── v0.40_topic06_claude_review_delta_20260619_KST.md
│   ├── v0.40_topic06_decision_log_draft_20260619_KST.md
│   └── v0.40_topic06_decision_log_20260619_HOLD_KST.md
├── scripts/
│   ├── README.md                               ← 실습 가이드 (Step1~4 수동 절차)
│   ├── rach_failure_prefilter.ps1               ← 키워드는 모듈명 아닌 증상/프로토콜 용어
│   └── scg_failure_prefilter.ps1                  (3차 세션에서 확인, 추정 키워드 다수 — 미검증)
└── rca_kg/                                     ← 실제 파일 기반 Knowledge Graph 트리
    ├── cases/
    │   ├── EXAMPLE_v2_rach_failure_001.yaml     ← ★ schema v2 적용 예시 (confirmed)
    │   └── unresolved/
    │       └── EXAMPLE_unresolved.yaml          ← ★ unresolved 예시 (handoff, null 필드)
    ├── indexes/index.md                          ← TODO: fingerprint 기준 재설계 필요
    ├── schema/
    │   ├── rca_case.schema.yaml                  ← ★★ v2 (3차 세션에서 전면 개정)
    │   └── taxonomy.yaml                         ← TODO: fingerprint 개념 반영 여부 확인 필요
    ├── skills_seed/rach_failure_analyzer.md
    ├── skills_seed/scg_failure_analyzer.md
    ├── skills_seed/tx_abnormal_analyzer.md
    ├── skills_seed/l2_max_retransmission_analyzer.md
    ├── skills_seed/crash_analyzer.md             ← ⚠ deprecated 확정됐으나 파일 미삭제 (TODO)
    └── signals/                                   ← 비어 있음 (pre-filter/L1SW 출력 저장 위치)
```

**포함하지 않은 것 (의도적 제외):** 5.1/5.2/5.3/5.4/5.6 등 RCA와 직접 관련 없는 L1a 다른
주제, L1a 전체 Master Roadmap, topic01~05 이력. RCA에 집중하기 위해 잘라냈다.

---

## 3. 세션 이력

### 3.1 1차 세션 (2026-06-19, claude.ai)

- 5.5 패키지를 L1a에서 분리, RCA standalone 트랙 생성
- L1SW Log Analyzer와의 비교 분석 착수 (`L1SW_LOG_ANALYZER_COMPARISON.md`)
- 미확인 5개 항목 도출

### 3.2 2차 세션 (2026-06-21, claude.ai)

- pre-filter와 5.5의 방향성 검토 수행
- **핵심 발견:**
  - pre-filter는 폐기 대상이 아님. L1SW(범용 필터)와 5.5 pre-filter(issue_type 특화 필터)는 계층이 다름
  - 5.5의 핵심 자산은 grep 스크립트가 아니라 **issue_type별 키워드 사전**
  - 키워드가 현재 3곳(scripts/ps1, skills_seed/md, prompt/md)에 분산 → **SSOT 필요** (`keywords.yaml` 제안)
- **산출물:** `L1SW_VERIFICATION_QUESTIONS_v2.md` (블록 A/B/C 15개 질문)

### 3.3 온사이트 확인 (2026-06-20, 사내 Claude Code 환경)

- 체크리스트(블록 A~F)에 대해 온사이트에서 직접 L1SW 스킬 파일을 확인
- **Option A 확정**: L1SW(1차 필터+HTML)와 5.5(유형분류+구조화+원인분석+자산연결)는 보완
  관계. 산출물: `L1SW_AND_FLOW_20260620.md`
- 핵심 확인 사항: L1SW manifest regex는 모듈명만 매핑, 라인번호 없음(timestamp 기반),
  issue_type 분류 없음, crash는 L1SW 전담

### 3.4 3차 세션 (2026-06-22, claude.ai) — 최신 ★★

`L1SW_AND_FLOW_20260620.md`를 출발점으로, "pre-filter가 결국 모듈별 키워드 아니냐"는
질문에서 시작해 **case 스키마 v2 전체 개정**까지 진행했다. 상세 설계 의도와 결정 이력은
`prompt/deepdive/RCA_FEEDBACK_LOOP_DESIGN_20260622_KST.md`에 전부 기록되어 있다 — 아래는
요지만 요약.

**핵심 발견·결정:**

1. **pre-filter 키워드의 정체**: 실제 스크립트(`rach_failure_prefilter.ps1` 등) 확인 결과,
   키워드는 모듈명이 아니라 증상/프로토콜 용어(`Msg1~4`, `RAR`, `T310`, `beamFail`). L1SW의
   모듈 축과는 다른 축 — `-Modules` 옵션으로 대체 불가.
2. **L1SW manifest 보강 방식 확정**: L1SW 코드를 수정하지 않고, issue_type 전용 fragment
   JSON(`rca_rach.json` 등)을 추가하는 방식으로 결론. `parse.ps1`의 `Select-String`이
   **줄 전체 매칭**임을 확인 → 증상 키워드를 manifest fragment에 그대로 넣어도 동작함이
   성립.
3. **case 용량 비대화 방지**: 로그 본문 인라인(`raw_examples`) 제거, `signal_file` +
   `cptime_range` 참조로 전환. 원칙은 "case는 포인터, 데이터는 한 곳에".
4. **동일 원인 재발 판정 — time_range 사용 불가**: 통신 프로토콜 로그는 단말 시험마다
   wall-clock이 다르므로 시각으로 "같은 장애"를 식별할 수 없음. → `fingerprint`
   (`signature_set` + `sequence`, 절대시각 미포함) 신설. 정렬 기준은 **cptime**으로 통일
   (wall-clock 드리프트 회피, 이벤트 선후관계는 시험 간 안정적).
5. **재발 처리**: `occurrence_count` + `recent_occurrences`(최근 5건만 유지)로 전환 —
   case 본체가 재발마다 증식하지 않음. `related.jira` 폐기, `recent_occurrences[].jira`로
   단일화(트레이드오프: 6번째 재발부터 오래된 Jira 참조 자연 소실, 의도적 수용).
6. **unresolved 재정의**: "원인 불명"이 아니라 **"담당영역 밖 → 타 담당자 이관"**으로 좁힘
   (원인 불확실은 `status:analyzed` + `confidence:low`로 이미 표현 가능, 겹치지 않음).
   `cases/unresolved/`에 별도 파일로 분리하고 `root_cause`/`fix`/`review`를 명시적
   `null`로 두어 "분석자 미입력"을 한눈에 식별 가능하게 함.
7. **통합(merge) 절차 확정**: 분석자가 unresolved 파일에 직접 입력 완료 후 fingerprint로
   기존 `cases/`와 매칭 → 일치 시 병합(occurrence_count++), 불일치 시 `cases/`로 승격.
8. **5.0/5.2 연결**: RCA 본 루프의 의존성으로 묶지 않고 `root_cause.code_ref` 옵션
   필드만 비워둠 — 5.2 산출물 구조 확인 후 TODO로 분리.

**산출물:**
- `rca_kg/schema/rca_case.schema.yaml` — v2 전면 개정 (위 1~8 모두 반영)
- `rca_kg/cases/EXAMPLE_v2_rach_failure_001.yaml` — confirmed 예시
- `rca_kg/cases/unresolved/EXAMPLE_unresolved.yaml` — unresolved 예시
- `prompt/deepdive/RCA_FEEDBACK_LOOP_DESIGN_20260622_KST.md` — 설계 의도·결정 이력 전문

---

## 4. 다음 세션에서 우선 진행할 것

### Phase 1 — schema v2 정합성 점검 (착수 권장)

1. `rca_kg/schema/taxonomy.yaml`이 schema v2의 `fingerprint`/`signature` 개념을 참조할
   필요가 있는지 확인 (현재 `root_cause_categories`만 정의되어 있음, signature 사전은
   아직 없음).
2. `rca_kg/indexes/index.md`를 fingerprint 기준 검색에 맞춰 재설계 (현재는 v1 구조 유지
   중으로 추정, 미확인).
3. `skills_seed/crash_analyzer.md` 처리 — deprecated가 이미 확정되었으나 파일이 패키지에
   남아있음. 삭제할지, 보존하되 명시적 deprecated 마크만 추가할지 결정.

### Phase 2 — keywords.yaml SSOT 구현

4. `keywords.yaml` 초안 작성 (issue_type별 confirmed/candidate/rejected 구조). 이때
   signature ID 체계가 schema v2의 `fingerprint.signature_set`,
   `log_patterns[].signature`와 동일 네임스페이스를 공유해야 함(SSOT 일관성 — 별도
   사전 분기 금지).
5. L1SW manifest fragment(`rca_rach.json` 등) 실제 작성 — 사내 L1SW 환경에서 진행.
   `rach_failure_prefilter.ps1`/`scg_failure_prefilter.ps1`의 "추정" 키워드를 1차 후보로
   사용하되, 실제 로그로 검증 필요(현재 `PHY_TIMER_EXPIRY`만 `# 확인됨`).

### Phase 3 — 검증

6. 실 로그 1건으로 end-to-end 시범 운영 — 특히 schema v2의 핵심 신규 메커니즘
   (fingerprint 자동 초안 정확도, unresolved→통합 절차, cptime 기반 sequence 정렬)을
   함께 검증.
7. 검증 결과로 keywords.yaml 첫 번째 업데이트 + manifest fragment 보정.

---

## 5. 이 트랙을 L1a로 다시 합류시키고 싶을 때

이 패키지의 `reference/master_v0.40_section_5_5.md`와 `prompt/`, `delta/`, `review_logs/`는
원본 L1a 파일과 동일한 내용이므로, RCA 설계가 안정화된 후 L1a Master Roadmap에 반영하고
싶다면 L1a 세션으로 돌아가 topic06을 재개(HOLD 해제)하면 된다. 그때는 이 패키지에서 갱신된
내용을 새 GPT Delta 또는 Claude Review Delta로 옮겨 L1a S4~S8 절차를 따른다.
