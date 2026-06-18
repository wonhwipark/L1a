# L1 AI Automation — L1a

GPT + Claude 협업 기반의 L1 AI 자동화 로드맵 문서 및 구현 산출물 저장소.

---

## 현재 상태

| 항목 | 내용 |
|------|------|
| 기준 Master | [`master/L1_AI_Automation_Roadmap_v0.40.md`](master/L1_AI_Automation_Roadmap_v0.40.md) |
| 진행 중 항목 | `5.5 RCA Knowledge Graph` — Step 1 완료 |
| 다음 단계 | 실제 로그로 pre-filter 실습 → RCA YAML 1건 저장 |

세부 진행 이력: [`last_status.md`](last_status.md)

---

## 디렉토리 구조

```
L1a/
  ├─ master/          기준 Master 문서 (Roadmap v0.38 ~ v0.40)
  ├─ prompt/          5.0~5.6 자동화 구현 프롬프트
  ├─ scripts/         로그 분석 PowerShell 스크립트
  ├─ rca_kg/          RCA Knowledge Graph (파일 기반)
  ├─ delta/           GPT Delta 산출물
  ├─ review_logs/     Claude Review / Decision / Merge Delta 로그
  ├─ readme_workflow.md   전체 협업 Workflow 운영 기준
  └─ last_status.md       현재 상태 및 다음 Step
```

---

## 빠른 시작 — 로그 분석 실습

> 자세한 절차: [`scripts/README.md`](scripts/README.md)

```powershell
# 1. PowerShell 에서 scripts 폴더로 이동
cd "d:\User\whpark\L1_AI_Automation\L1a\scripts"

# 2. RACH failure pre-filter 실행
.\rach_failure_prefilter.ps1 -InputLog "C:\경로\service.log" `
    -OutputTxt "..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt"

# 3. Claude Code 에 분석 요청 (채팅창에 붙여넣기)
# @rca_kg/signals/2026-06-20_rach_failure_001_signal.txt
# @rca_kg/skills_seed/rach_failure_analyzer.md
# @rca_kg/cases/EXAMPLE_rach_failure_001.yaml
# → 분석 후 cases/ 에 YAML 저장 + indexes/index.md 업데이트 요청
```

---

## 주요 파일 안내

| 파일/폴더 | 용도 |
|-----------|------|
| [`master/L1_AI_Automation_Roadmap_v0.40.md`](master/L1_AI_Automation_Roadmap_v0.40.md) | 전체 자동화 로드맵 기준 문서 |
| [`prompt/5_5_rca_knowledge_graph_py_package_prompt.md`](prompt/5_5_rca_knowledge_graph_py_package_prompt.md) | 5.5 RCA KG 구현 프롬프트 (Step1/Step2 정의) |
| [`scripts/rach_failure_prefilter.ps1`](scripts/rach_failure_prefilter.ps1) | RACH failure 로그 pre-filter |
| [`scripts/scg_failure_prefilter.ps1`](scripts/scg_failure_prefilter.ps1) | SCG failure 로그 pre-filter |
| [`rca_kg/cases/EXAMPLE_rach_failure_001.yaml`](rca_kg/cases/EXAMPLE_rach_failure_001.yaml) | RCA YAML 작성 템플릿 |
| [`rca_kg/schema/rca_case.schema.yaml`](rca_kg/schema/rca_case.schema.yaml) | YAML 필드 정의 |
| [`rca_kg/schema/taxonomy.yaml`](rca_kg/schema/taxonomy.yaml) | issue_type / confidence 기준 |
| [`rca_kg/indexes/index.md`](rca_kg/indexes/index.md) | 전체 케이스 목록 인덱스 |
| [`readme_workflow.md`](readme_workflow.md) | GPT-Claude 협업 Workflow 기준 |

---

## RCA Knowledge Graph 구조

분석 1건 = YAML 1개. Claude Code 가 자동 생성.

```
rca_kg/
  ├─ signals/         pre-filter 출력 (log → 수 MB 로 축소)
  ├─ cases/           RCA 분석 결과 YAML (Claude 자동 생성)
  ├─ skills_seed/     문제 유형별 분석 checklist (반복 시 누적)
  ├─ indexes/         케이스 목록 인덱스
  └─ schema/          YAML 스키마 + taxonomy
```

지원 문제 유형: `rach_failure` / `scg_failure` / `tx_abnormal` / `l2_retx` / `crash`

---

## 작업 흐름

1. 기준 Master 확인 (`last_status.md`)
2. GPT 가 주제 논의 및 Delta 초안 작성 (`delta/`)
3. Claude 가 Review Delta 및 Decision Log 작성 (`review_logs/`)
4. 사용자 수용/기각 판단
5. Claude 가 Merge Delta 및 Master 갱신
6. 상태 갱신 (`last_status.md`)
