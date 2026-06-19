# L1SW Log Analyzer vs 5.5 RCA Knowledge Graph — 비교 분석

**작성:** Claude (claude.ai 채팅 세션, L1a 패키지 검토 중)
**일시:** 2026-06-19 KST
**배경:** 사용자가 사내에서 L1SW Log Analyzer 스킬을 전달받아 사용 중이며, 5.5 RCA Knowledge Graph와의 상충 여부를 확인하고자 함. 이 검토 과정에서 RCA를 L1a로부터 별도 트랙으로 분리하기로 결정함.

---

## 1. 사용자가 제공한 L1SW Log Analyzer 정보 (원문 정리)

```text
이름: l1sw-log-analyzer

입력 파이프라인:
  .sdm live trace
    → parse.ps1
    → _l1sw.txt 생성
    → 분석

토큰 절약 옵션:
  -TimeFrom, -TimeTo   (시간 윈도우)
  -Modules             (모듈 부분집합, 예: 'Allocator, DSSM')

출력:
  Self-contained HTML 리포트
    - CSS 인라인
    - 다크 헤더
    - 색상 배지(severity 등으로 추정)
  로컬 파일로 저장

필터링 메커니즘 (Single Source of Truth):
  Manifest 디렉터리의 5개 fragment JSON
    → 각 JSON: 모듈명 → regex 매핑 구조
    → sdm parser에 전달되어 prefix/키워드 기반 필터링 수행

모듈 분기:
  -Modules 옵션으로 부분집합 필터 가능 (예: 'Allocator, DSSM')
  [DSSM] 같은 모듈 prefix로 분기 가능
```

**확인 완료 (2026-06-19 KST):**
1. `_l1sw.txt`는 원본 `.sdm` 대비 약 **80% 축소** (원본의 ~20% 수준)
2. 초기 분석에서는 Context 초과 리스크 있음 (성능 저하), 정보 누적으로 점진 개선
3. 과거 HTML 결과 참조/재사용 메커니즘 **없음**
4. issue_type 분류 체계 **없음** — 모듈 기반 분류만 존재
5. 구조화 데이터(YAML/JSON) 병행 저장 **없음** — HTML only

**추가 확인 (2026-06-20 확인 예정):**
- L1SW에 지식 누적 메커니즘 존재 — skill 보강 방식으로 추정. 상세 구조 확인 필요.

**확정 사항:**
- **crash 분석은 L1SW 전담.** dump 파일 기반 별도 분석 기능이 L1SW에 이미 있으며, 그대로 사용 예정. 5.5의 crash issue_type(`crash_analyzer.md`, taxonomy 내 crash 관련 root_cause)은 폐기 또는 L1SW 결과 참조용으로 축소 검토.

---

## 2. 비교표 (확정)

| 항목 | L1SW Log Analyzer | 5.5 rca_kg/ (L1a) |
|------|---|---|
| 입력 | `.sdm` live trace → `parse.ps1` → `_l1sw.txt` (80% 축소) | raw `log.txt` (100MB~1GB) → issue-type pre-filter → `signal_<issue_type>.txt` |
| 필터링 기준 | Manifest 기반 모듈명→regex 매핑 (5개 fragment JSON), prefix/키워드 | Issue type별 keyword/signature (rach/scg/tx/l2/crash) |
| 토큰 절약 방식 | 시간윈도우(`-TimeFrom/-TimeTo`) + 모듈 부분집합(`-Modules`) | Context window(전후 N줄) + issue-type 필터 |
| 분기 기준 | 모듈명 (`[DSSM]`, `Allocator` 등) | 문제 유형 (issue_type) |
| 출력 형식 | Self-contained HTML (사람이 읽는 리포트) | case YAML + RCA Markdown (LLM/Git 친화적 구조화 데이터) |
| 저장 방식 | 로컬 HTML 파일 | `rca_kg/cases/*.yaml` + `indexes/` + `skills_seed/*.md` |
| 재사용 구조 | **있음** (skill 보강 방식 추정, 6/20 확인 예정) | case YAML 누적 → index → skills_seed 패턴 정제 |
| 문제유형 분류 | **없음** (모듈 기반만) | issue_type taxonomy (5종) + root_cause_category (10종) |
| 구조화 저장 | **없음** (HTML only) | YAML + Markdown |
| 과거 결과 참조 | **없음** | case index → 유사 케이스 검색 |
| crash 분석 | **있음** (dump 파일 기반 별도 기능) — 전담 | crash_analyzer.md 있으나 L1SW에 위임 예정 |

---

## 3. 확정 판단

5개 항목 확인 결과, L1SW와 5.5는 **중복이 아니라 상호 보완 관계**임이 확인되었다.

**L1SW가 갖고 있지 않은 것 (= 5.5가 채우는 공백):**
- 문제유형(issue_type) 분류 체계 → L1SW는 모듈 기반만
- 구조화 데이터 저장 → L1SW는 HTML only
- 과거 분석 결과 참조/재사용 → L1SW는 HTML 참조 없음 (단, skill 보강으로 지식 누적은 있음 — 6/20 확인 예정)

**L1SW가 잘하는 것 (= 5.5가 중복할 필요 없는 것):**
- `.sdm` → `_l1sw.txt` 파싱 (80% 축소) — 이미 검증된 파이프라인
- 모듈/시간 기반 1차 필터링
- 사람이 보는 HTML 리포트 생성
- skill 보강을 통한 분석 품질 점진 개선 (상세 구조 확인 필요)
- **crash 분석 전담** — dump 파일 기반 별도 기능, 5.5에서 중복 불필요

**→ 5.5 scope 조정:** issue_type 5종 → **4종** (rach_failure, scg_failure, tx_abnormal, l2_max_retransmission). crash는 L1SW 전담.

**결론:** Option A(역할 분리)가 여전히 유력하나, L1SW의 skill 보강 구조 확인 후
5.5 `skills_seed`와의 중복 범위를 판단해야 최종 확정 가능. crash는 L1SW 전담 확정,
5.5는 4종(rach/scg/tx/l2)에 집중. 6/20 skill 보강 확인 후 결정.

---

## 4. 통합/분리 방향 옵션 (다음 세션에서 결정)

| 옵션 | 내용 |
|------|------|
| A | L1SW가 1차 필터(모듈/시간) 담당, 5.5는 그 결과(`_l1sw.txt`)를 입력으로 받아 issue-type 분류 + RCA YAML 누적만 담당 (역할 분리, 5.5의 pre-filter 단계 폐기) |
| B | 두 파이프라인 독립 유지, 5.5는 L1SW를 거치지 않은 원시 로그도 처리 가능하게 유지 (병렬 운영) |
| C | 5.5의 case YAML 스키마에 L1SW HTML 리포트 경로/요약을 `source` 필드로 연결만 하고 나머지는 그대로 (느슨한 연동) |

---

## 5. 다음 액션

1. ~~사내 Claude Code 환경에서 L1SW Log Analyzer 스킬 파일을 직접 읽어 위 5개 미확인 항목 보완~~ → **완료**
2. **6/20 확인 예정:** L1SW skill 보강 구조 상세 — 5.5 `skills_seed`와 중복 범위 판단
3. 확인 후 옵션 A/B/C 최종 확정
4. Option 확정 시 → `prompt/5_5_..._prompt.md` 재정의 + crash 관련 자산 정리 (taxonomy에서 crash 관련 root_cause 제거 또는 L1SW 참조용으로 축소, `crash_analyzer.md` 폐기)
5. 실 로그 1건으로 end-to-end 시범 운영
