# L1a Repository Overview

이 저장소는 [`GPT`](L1a/README.md:3) 와 [`Claude`](L1a/README.md:3) 협업 기반의 문서 버전업 및 프롬프트 운영 산출물을 관리하는 작업 저장소다.

## 구조

- [`master/`](L1a/master): 기준 Master 문서 보관 디렉터리
- [`prompt/`](L1a/prompt): 구현/운영 프롬프트 문서 디렉터리
- [`delta/`](L1a/delta): GPT 단계의 변경 Delta 산출물 디렉터리
- [`review_logs/`](L1a/review_logs): Claude 검토, Merge, Decision Log 산출물 디렉터리
- [`readme_workflow.md`](L1a/readme_workflow.md): 전체 협업 Workflow의 단일 운영 기준 문서
- [`last_status.md`](L1a/last_status.md): 현재 기준 Master, 최근 완료 Step, 다음 Step 상태 문서

## 현재 상태

- 현재 기준 Master: [`master/L1_AI_Automation_Roadmap_v0.39.md`](L1a/master/L1_AI_Automation_Roadmap_v0.39.md)
- 현재 Topic: 없음
- 다음 Step: [`S0`](L1a/readme_workflow.md:18) — 새 topic 선정 후 GPT와 주제별 논의 시작

세부 상태와 진행 이력은 [`last_status.md`](L1a/last_status.md) 를 우선 확인한다.

## 작업 흐름 요약

이 저장소는 [`readme_workflow.md`](L1a/readme_workflow.md) 에 정의된 협업 흐름을 기준으로 운영된다.

1. 기준 Master 확인
2. GPT가 주제 논의 및 Delta 초안 작성
3. Claude가 Review Delta 및 Decision Log 초안 작성
4. 사용자 수용/기각 판단 반영
5. Claude가 Merge Delta 및 필요 시 New Master 생성
6. Decision Log 확정 및 상태 갱신

## 사용 원칙

- 기준 문서는 직접 덮어쓰기보다 Delta 중심으로 변경한다.
- 최신 진행 상태 재개 시 [`last_status.md`](L1a/last_status.md) 를 먼저 확인한다.
- 운영 절차와 Step 정의는 [`readme_workflow.md`](L1a/readme_workflow.md) 를 단일 출처로 사용한다.
