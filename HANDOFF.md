# RCA Standalone — Handoff

이 패키지는 **L1a Master Roadmap의 5.5 RCA Knowledge Graph**를 L1a 본 워크플로(S0~S9)에서
분리하여 별도 트랙으로 진행하기 위해 추출한 것이다.

새 세션에서는 이 파일을 가장 먼저 읽는다.

---

## ⏩ 2026-06-20 새 세션 빠른 시작 (최신)

직전 세션(2026-06-19 저녁)에서 "5.5가 L1SW에서 못 채우는 부분을 더 보강하는 방향"을 논의했다.
새 창에서 이어서 진행할 때 아래 순서로 읽는다:

1. **`DISCUSSION_20260619_2105_KST.md`** ← 직전 논의 결론. L1SW 공백 7개 영역 전수 분석 +
   보강 우선순위 (미확정, 6/20 확인 대기 상태).
2. **`L1SW_QUESTIONS_20260620.md`** ← 6/20에 L1SW를 직접 확인하며 채울 질문지 (A~F 블록, A 최우선).
   답을 채운 뒤 새 세션에 올리면 보강 방향이 확정된다.

**현재 상태:** 보강 방향 논의는 했으나, 사용자가 L1SW를 아직 실사용하지 못해 7개 영역의
경계가 추정 상태. 6/20 L1SW 직접 확인 → 답 기반으로 ①살릴 것/접을 것 + ②Option A/B/C 확정.

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
├── L1SW_LOG_ANALYZER_COMPARISON.md             ← L1SW vs 5.5 비교 분석 (직전 세션 산출물)
├── reference/
│   └── master_v0.40_section_5_5.md             ← L1a Master §5.5 원문 발췌 (참조용 사본)
├── prompt/
│   └── 5_5_rca_knowledge_graph_py_package_prompt.md   ← 5.5 구현 프롬프트 (Step1/Step2 정의)
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

### 의도적 제외

5.1/5.2/5.3/5.4/5.6 등 RCA와 직접 관련 없는 L1a 다른 주제, L1a 전체 Master Roadmap,
topic01~05 이력은 포함하지 않았다. RCA에 집중하기 위해 잘라냈다.

---

## 3. 확인된 사항 (2026-06-19 claude.ai 세션 2회 누적)

### 3.1 5.5 실파일 상태

- 설계와 실파일(schema, skills_seed 5종, example case, pre-filter script 2종)이 갖춰져 있으나,
  **실 로그로 end-to-end 검증을 한 번도 하지 않은 상태.**
- pre-filter script는 rach_failure, scg_failure 2종만 있고 tx_abnormal/l2_max_retransmission은
  미작성. crash는 L1SW 전담 확정으로 5.5에서 폐기 예정.
- `issue_type` 표준명은 `l2_max_retransmission`으로 확정. `rca_kg/` 실파일은 이미 통일됨.

### 3.2 L1SW Log Analyzer 확인 결과

사용자가 사내에서 사용 중인 L1SW Log Analyzer의 정보를 확인했다. 상세는
`L1SW_LOG_ANALYZER_COMPARISON.md`에 기록되어 있으며, 여기서는 핵심만 정리한다.

**L1SW 개요:**
```text
이름: l1sw-log-analyzer
스킬 위치: 루트의 .claude/ 디렉터리

파이프라인:
  .sdm live trace → parse.ps1 → _l1sw.txt 생성 (원본 대비 80% 축소) → 분석

토큰 절약 옵션:
  -TimeFrom, -TimeTo   (시간 윈도우)
  -Modules             (모듈 부분집합, 예: 'Allocator, DSSM')

출력:
  Self-contained HTML 리포트 (CSS 인라인, 다크 헤더, 색상 배지)
  로컬 파일 저장

필터링 (Single Source of Truth):
  Manifest 디렉터리의 5개 fragment JSON
    → 각 JSON: 모듈명 → regex 매핑 구조
    → sdm parser에 전달되어 prefix/키워드 기반 필터링 수행

모듈 분기:
  -Modules 옵션으로 부분집합 필터 가능 (예: 'Allocator, DSSM')
  [DSSM] 같은 모듈 prefix로 분기 가능

crash 분석:
  dump 파일 기반 별도 분석 기능이 추가 기능으로 존재
  → crash는 L1SW 전담 확정

지식 누적:
  있음 — skill 보강 방식으로 추정 (6/20 상세 확인 예정)
```

**확인 완료 5개 항목:**
1. `_l1sw.txt`는 원본 `.sdm` 대비 약 80% 축소 (원본의 ~20% 수준)
2. 초기 분석에서는 Context 초과 리스크 있음 (성능 저하), 정보 누적으로 점진 개선
3. 과거 HTML 결과 참조/재사용 메커니즘 없음
4. issue_type 분류 체계 없음 — 모듈 기반 분류만 존재
5. 구조화 데이터(YAML/JSON) 병행 저장 없음 — HTML only

**미확인 (6/20 확인 예정):**
- L1SW skill 보강의 상세 구조 (모듈별? 이슈 패턴별? 어떤 형식으로 누적?)
- 5.5 `skills_seed`와의 중복 범위

### 3.3 1차 판단: L1SW vs 5.5 관계

**상호 보완 관계** 확인. 중복이 아님.

- L1SW에 없는 것 (= 5.5가 채우는 공백): issue_type 분류, 구조화 저장(YAML), 과거 케이스
  참조/검색, taxonomy
- L1SW가 잘하는 것 (= 5.5가 중복할 필요 없는 것): `.sdm` 파싱/축소, 모듈/시간 필터링,
  HTML 리포트, crash 분석(dump 기반)

**5.5 scope 조정:** issue_type 5종 → 4종 (crash 제외)
**5.5 위치 재정의:** "L1SW가 1차로 줄여준 로그를 받아서 문제유형 분류 + 구조화 저장 +
지식 누적을 하는 후처리 계층"

**핵심 개념은 변하지 않음.** 입력 경로(raw `.sdm` → L1SW `_l1sw.txt`)와 scope(5종→4종)만
조정. Option A(역할 분리)가 유력하나, L1SW skill 보강 확인 후 최종 확정.

---

## 4. 다음 세션에서 진행할 것

### 즉시 진행 가능 (L1SW skill 보강 확인 전이라도)

1. **Option A/B/C 방향 논의** — `L1SW_LOG_ANALYZER_COMPARISON.md` §3~4 기반.
   현재 Option A(역할 분리)가 유력. crash L1SW 전담은 확정.

2. **5.5 설계 갱신 착수** — Option A 기준:
   - `prompt/5_5_..._prompt.md`의 Step 1 pre-filter → L1SW `_l1sw.txt` 입력 구조로 재정의
   - taxonomy.yaml에서 crash 관련 항목 정리 (제거 또는 L1SW 참조용 축소)
   - `skills_seed/crash_analyzer.md` 폐기
   - issue_type 5종 → 4종 반영

### L1SW skill 보강 확인 후 (6/20 예정)

3. **L1SW skill 보강 구조 확인** — 사내 Claude Code에서 `.claude/` 스킬 파일 직접 확인.
   5.5 `skills_seed`와 중복 범위 판단.
4. 중복 범위에 따라 `skills_seed` 역할 조정 (L1SW가 모듈 기반 skill만 보강한다면 5.5
   skills_seed는 issue_type 기반이므로 겹치지 않아 현재 설계 유지 가능)

### 설계 확정 후

5. **실 로그 1건으로 end-to-end 시범 운영**
   L1SW `_l1sw.txt` → issue-type 분류 → RCA 분석 → case YAML → index 업데이트

---

## 5. 이 트랙을 L1a로 다시 합류시키고 싶을 때

이 패키지의 `reference/master_v0.40_section_5_5.md`와 `prompt/`, `delta/`, `review_logs/`는
원본 L1a 파일과 동일한 내용이므로, RCA 설계가 안정화된 후 L1a Master Roadmap에 반영하고
싶다면 L1a 세션으로 돌아가 topic06을 재개(HOLD 해제)하면 된다. 그때는 이 패키지에서 갱신된
내용을 새 GPT Delta 또는 Claude Review Delta로 옮겨 L1a S4~S8 절차를 따른다.
