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
│       ├── L1SW_LOG_ANALYZER_COMPARISON.md      ← L1SW vs 5.5 비교 분석
│       ├── L1SW_VERIFICATION_QUESTIONS_v2.md    ← L1SW 추가 확인 질문 15개 (서술형, 배경 참조용)
│       ├── L1SW_CHECKLIST_OBJECTIVE_v2.md       ← ★ 같은 15개를 객관식으로 재구성 (현장용)
│       └── session_log_20260621_KST.md          ← 방향성 검토 세션 논의 요약
├── delta/
│   └── v0.40_topic06_gpt_delta_..._20260619_KST.md    ← topic06 GPT Delta (HOLD된 확장 제안)
├── review_logs/
│   ├── v0.40_topic06_claude_review_delta_20260619_KST.md
│   ├── v0.40_topic06_decision_log_draft_20260619_KST.md
│   └── v0.40_topic06_decision_log_20260619_HOLD_KST.md
├── scripts/
│   ├── README.md                               ← 실습 가이드 (Step1~4 수동 절차)
│   ├── rach_failure_prefilter.ps1
│   └── scg_failure_prefilter.ps1
└── rca_kg/                                     ← 실제 파일 기반 Knowledge Graph 트리
    ├── cases/EXAMPLE_rach_failure_001.yaml      ← 출력 형식 예시
    ├── indexes/index.md
    ├── schema/rca_case.schema.yaml
    ├── schema/taxonomy.yaml
    ├── skills_seed/rach_failure_analyzer.md
    ├── skills_seed/scg_failure_analyzer.md
    ├── skills_seed/tx_abnormal_analyzer.md
    ├── skills_seed/l2_max_retransmission_analyzer.md
    ├── skills_seed/crash_analyzer.md
    └── signals/                                 ← 비어 있음 (pre-filter 출력 저장 위치)
```

**포함하지 않은 것 (의도적 제외):** 5.1/5.2/5.3/5.4/5.6 등 RCA와 직접 관련 없는 L1a 다른
주제, L1a 전체 Master Roadmap, topic01~05 이력. RCA에 집중하기 위해 잘라냈다.

---

## 3. 세션 이력

### 3.1 1차 세션 (2026-06-19, claude.ai)

- 5.5 패키지를 L1a에서 분리, RCA standalone 트랙 생성
- L1SW Log Analyzer와의 비교 분석 착수 (`L1SW_LOG_ANALYZER_COMPARISON.md`)
- 미확인 5개 항목 도출

### 3.2 2차 세션 (2026-06-21, claude.ai) — 최신

- pre-filter와 5.5의 방향성 검토 수행
- **핵심 발견:**
  - pre-filter는 폐기 대상이 아님. L1SW(범용 필터)와 5.5 pre-filter(issue_type 특화 필터)는 계층이 다름
  - 5.5의 핵심 자산은 grep 스크립트가 아니라 **issue_type별 키워드 사전**
  - 키워드가 현재 3곳(scripts/ps1, skills_seed/md, prompt/md)에 분산 → **SSOT 필요** (`keywords.yaml` 제안)
  - 키워드는 고정이 아니라 분석 누적을 통해 **confirmed/rejected/candidate로 진화**해야 함
  - L1SW의 키워드 관리 구조를 정확히 파악해야 5.5와의 역할 경계를 확정할 수 있음
- **산출물:** `L1SW_VERIFICATION_QUESTIONS_v2.md` (블록 A/B/C 15개 질문)
- 상세: `session_log_20260621_KST.md` 참조

---

## 4. 다음 세션에서 우선 진행할 것

### Phase 1 — L1SW 질문 답변 수집 (사내 Claude Code 환경)

1. `prompt/deepdive/L1SW_CHECKLIST_OBJECTIVE_v2.md`(객관식)를 열고, 사내 Claude Code에서
   L1SW 스킬 파일을 `@mention`하거나 bash로 읽어 블록 A(5개), B(6개), C(4개) 총 15개
   질문에 번호만 표기한다. 배경이 필요하면 `L1SW_VERIFICATION_QUESTIONS_v2.md`(서술형) 참조.
2. 특히 **블록 B-3**(키워드와 issue_type 매핑 관계)과 **블록 C-3**(스킬 강화 메커니즘)의
   답변이 5.5 설계 방향을 결정하는 핵심이다.

### Phase 2 — 답변 기반 설계 확정

3. 답변 결과에 따라 Option A/B/C 최종 선택
4. `keywords.yaml` SSOT 파일 초안 작성 (issue_type별 confirmed/candidate/rejected 구조)
5. 키워드 누적 개선 루프 설계 확정
6. 5.5 입력 소스를 `_l1sw.txt`로 재정의 (Option A 채택 시, pre-filter 스크립트는
   raw log가 아닌 `_l1sw.txt`를 입력으로 받도록 수정)
7. `5_5_..._prompt.md` 분리 여부 결정 (구현 명세 vs 운영 가이드)

### Phase 3 — 검증

8. 실 로그 1건으로 end-to-end 시범 운영
9. skills_seed "추정" 키워드 실검증 → keywords.yaml 첫 번째 업데이트

### 답변 결과별 예상 분기 (참고)

| 답변 결과 | 5.5 설계 영향 |
|---|---|
| B-3: L1SW에 issue_type 매핑 없음 | → `keywords.yaml`이 유일한 SSOT |
| B-3: L1SW에 issue_type 매핑 있음 | → `keywords.yaml`은 L1SW 매핑을 확장/보완 |
| B-4: 모듈 필터만으로 충분 | → 5.5 pre-filter 2차 grep 불필요할 수 있음 |
| B-4: 모듈 필터 결과에 잡음 많음 | → 5.5 pre-filter 2차 grep 필수 (현재 설계 유지) |
| C-3: 스킬 강화가 모듈 단위 | → `skills_seed/`와 직교, 병행 운영 |
| C-3: 스킬 강화가 이슈 단위 | → `skills_seed/`와 겹침, 역할 재정의 필요 |

---

## 5. 이 트랙을 L1a로 다시 합류시키고 싶을 때

이 패키지의 `reference/master_v0.40_section_5_5.md`와 `prompt/`, `delta/`, `review_logs/`는
원본 L1a 파일과 동일한 내용이므로, RCA 설계가 안정화된 후 L1a Master Roadmap에 반영하고
싶다면 L1a 세션으로 돌아가 topic06을 재개(HOLD 해제)하면 된다. 그때는 이 패키지에서 갱신된
내용을 새 GPT Delta 또는 Claude Review Delta로 옮겨 L1a S4~S8 절차를 따른다.
