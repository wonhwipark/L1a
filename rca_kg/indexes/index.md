# RCA Knowledge Graph — Case Index

> Phase 1 index: 수동 관리. 10건 이후 by_issue_type.yaml, by_root_cause.yaml 추가 예정.

---

## 사용법

새 케이스 분석 완료 시 아래 표에 한 줄 추가한다.

```
| YYYY-MM-DD | case_id | issue_type | root_cause category | confidence | status |
```

유사 케이스 검색 시 이 표에서 issue_type / root_cause 로 먼저 필터링한 뒤,
후보 case YAML 2~3개만 Claude/Roo 에 넣는다.

---

## Case 목록

| 날짜 | case_id | issue_type | root_cause | confidence | status |
|------|---------|------------|------------|------------|--------|
| -    | EXAMPLE_rach_failure_001 (템플릿) | rach_failure | radio_access_timeout | medium | draft |

---

## 통계 요약 (수동 업데이트)

| issue_type | 건수 |
|------------|------|
| rach_failure | 0 |
| scg_failure | 0 |
| tx_abnormal | 0 |
| l2_retx | 0 |
| crash | 0 |
| **합계** | **0** |

---

## 디렉토리 구조

```
rca_kg/
  ├─ cases/           ← 분석 1건당 YAML 1개
  ├─ signals/         ← pre-filter 출력 signal 파일
  ├─ skills_seed/     ← 문제 유형별 분석 가이드 (반복 분석 시 업데이트)
  ├─ indexes/
  │    └─ index.md    ← 이 파일 (Phase 1 전체 인덱스)
  └─ schema/
       ├─ rca_case.schema.yaml   ← 필드 정의
       └─ taxonomy.yaml          ← issue_type / root_cause / confidence 기준
```
