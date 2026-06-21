# RCA Standalone 로그 분석 운영 가이드 — schema v2

> 대상: `rach_failure`, `scg_failure`, `tx_abnormal`, `l2_max_retransmission`  
> 제외: `crash` / dump 분석은 L1SW Log Analyzer 전담  
> 환경: Windows 11, PowerShell, VSCode + Claude Code 또는 Roo Code  
> 기준 스키마: `rca_kg/schema/rca_case.schema.yaml` v2  
> 현재 단계: R3 `keywords.yaml` SSOT v0.1 생성 완료. 다음 단계는 R4 L1SW manifest fragment 후보 설계

---

## 1. 이 가이드의 목적

이 문서는 대용량 L1 로그를 RCA Knowledge Graph case YAML로 누적하기 위한 수동 운영 가이드다.

핵심 원칙은 아래와 같다.

```text
원본 로그 전체를 AI Context에 직접 넣지 않는다.
L1SW Log Analyzer로 1차 축소한 _l1sw.txt를 우선 입력으로 사용한다.
필요한 경우 issue_type별 signal 파일을 추가로 만든다.
RCA 결과는 schema v2 case YAML로 저장한다.
동일 원인은 fingerprint로 매칭하고 신규 case를 만들지 않는다.
```

---

## 2. 전체 흐름

```text
① .sdm 원본 로그
      ↓  [사람 / 사내 L1SW]
② L1SW Log Analyzer 실행
      ↓
③ _l1sw.txt 생성
      ↓  [선택: scripts/*.ps1]
④ issue_type별 signal 파일 생성
      ↓  [Claude Code 또는 Roo Code]
⑤ RCA 분석 + fingerprint draft 생성
      ↓
⑥ 기존 cases/와 fingerprint 매칭
      ├─ 일치: 기존 case occurrence_count 증가 + recent_occurrences 추가
      ├─ 불일치: 신규 fingerprint 기반 case YAML 생성
      └─ 담당영역 밖: cases/unresolved/에 PENDING YAML 생성
      ↓  [사람]
⑦ review / sequence_status / root_cause 확인
      ↓
⑧ index.md, skills_seed, 향후 keywords.yaml 후보 업데이트
```

---

## 3. Active issue_type 기준

현재 5.5 RCA KG에서 신규 case 생성 대상은 아래 4개다.

| issue_type | 설명 | 사용 skills_seed |
|---|---|---|
| `rach_failure` | RACH 절차 실패 | `rca_kg/skills_seed/rach_failure_analyzer.md` |
| `scg_failure` | SCG 추가/설정 실패 또는 RLF | `rca_kg/skills_seed/scg_failure_analyzer.md` |
| `tx_abnormal` | UL 송신 이상, UL grant 이상, HARQ fail 연속 | `rca_kg/skills_seed/tx_abnormal_analyzer.md` |
| `l2_max_retransmission` | RLC/HARQ 최대 재전송 한계 도달 | `rca_kg/skills_seed/l2_max_retransmission_analyzer.md` |

`crash`는 active issue_type이 아니다.

```text
crash / assert / fatal / dump 분석은 L1SW Log Analyzer 전담이다.
5.5 RCA KG에서는 crash case YAML을 신규 생성하지 않는다.
필요한 경우 L1SW HTML 또는 dump 분석 산출물 경로만 참조한다.
```

---

## 4. 준비물 체크리스트

시작 전 아래를 확인한다.

- [ ] 분석할 `.sdm` 원본 로그 또는 이미 생성된 `_l1sw.txt`가 있다.
- [ ] 사내 L1SW Log Analyzer를 실행할 수 있다.
- [ ] VSCode에서 이 `RCA_standalone` 폴더를 열 수 있다.
- [ ] Claude Code 또는 Roo Code가 로컬 파일을 읽고 쓸 수 있다.
- [ ] PowerShell 터미널을 열 수 있다.
- [ ] 아래 기준 파일이 존재한다.

```text
HANDOFF.md
rca_kg/schema/rca_case.schema.yaml
rca_kg/schema/taxonomy.yaml
rca_kg/indexes/index.md
rca_kg/cases/EXAMPLE_v2_rach_failure_001.yaml
rca_kg/cases/unresolved/EXAMPLE_unresolved.yaml
```

---

## 5. Step 1 — L1SW로 1차 축소본 생성

이 단계는 사내 L1SW Log Analyzer 환경에서 수행한다.

```text
.sdm 원본
  → L1SW parse.ps1
  → _l1sw.txt
```

중요 기준:

```text
_l1sw.txt에는 line number가 없다고 가정한다.
위치 식별은 line_range가 아니라 cptime_range를 사용한다.
_l1sw.txt는 삭제하지 않고 보존한다.
RCA case YAML에는 로그 본문을 복사하지 않고 signal_file 경로와 cptime_range만 기록한다.
```

---

## 6. Step 2 — 선택 사항: signal 파일 생성

L1SW가 만든 `_l1sw.txt`가 너무 크면 issue_type별 pre-filter 스크립트로 signal 파일을 만든다.

현재 `scripts/*.ps1`은 아직 `keywords.yaml`에서 자동 생성되지 않는 임시 수동 도구다.
키워드 SSOT는 `rca_kg/keywords.yaml`이며, 스크립트 내부 `$keywords` 블록은 R4 이후 동기화 대상이다.

### 6.1 PowerShell 위치 이동

VSCode 터미널 또는 PowerShell에서 `RCA_standalone/scripts` 폴더로 이동한다.

```powershell
cd "<RCA_standalone_경로>\scripts"
```

예시:

```powershell
cd "D:\work\RCA_standalone\scripts"
```

### 6.2 RACH failure signal 생성

입력은 원본 `.sdm`이 아니라 L1SW가 만든 `_l1sw.txt`를 권장한다.

```powershell
.\rach_failure_prefilter.ps1 `
  -InputLog "D:\work\logs\dump001_l1sw.txt" `
  -OutputTxt "..\rca_kg\signals\20260622_rach_failure_dump001_signal.txt" `
  -Context 20
```

### 6.3 SCG failure signal 생성

```powershell
.\scg_failure_prefilter.ps1 `
  -InputLog "D:\work\logs\dump001_l1sw.txt" `
  -OutputTxt "..\rca_kg\signals\20260622_scg_failure_dump001_signal.txt" `
  -Context 20
```

### 6.4 signal 파일명 규칙

signal 파일은 임시 분석 입력이므로 날짜 기반 이름을 사용할 수 있다.
단, case_id는 날짜 기반으로 만들지 않는다.

권장 signal 파일명:

```text
rca_kg/signals/<YYYYMMDD>_<issue_type>_<source-slug>_signal.txt
```

예시:

```text
rca_kg/signals/20260622_rach_failure_dump001_signal.txt
rca_kg/signals/20260622_scg_failure_dump007_signal.txt
```

case YAML의 정식 `case_id`는 분석 후 fingerprint 기반으로 생성한다.

```text
<fingerprint-slug>_<issue_type>_<3-digit-seq>
```

예시:

```text
rach_msg3timeout_rach_failure_001
scg_beamfail_scg_failure_001
```

---

## 7. Step 3 — signal 파일 확인

PowerShell에서 signal 파일이 생성됐는지 확인한다.

```powershell
ls ..\rca_kg\signals\
```

첫 부분을 확인한다.

```powershell
Get-Content "..\rca_kg\signals\20260622_rach_failure_dump001_signal.txt" -TotalCount 30
```

줄 수를 확인한다.

```powershell
(Get-Content "..\rca_kg\signals\20260622_rach_failure_dump001_signal.txt" | Measure-Object -Line).Lines
```

판단 기준:

| signal 줄 수 | 판단 | 조치 |
|---:|---|---|
| 0 | 키워드 미매칭 | 스크립트 키워드 후보를 실제 로그 기준으로 점검 |
| 1~100 | 너무 적을 수 있음 | `-Context 50`으로 재실행 검토 |
| 1,000~50,000 | 1차 분석 가능 | Claude Code 분석 진행 |
| 100,000 이상 | 너무 클 수 있음 | `-Context 5` 또는 L1SW time window 사용 검토 |

---

## 8. Step 4 — Claude Code / Roo Code 분석 요청

아래 프롬프트를 사용한다. `issue_type`, signal 파일명, 저장 파일명 후보만 상황에 맞게 바꾼다.

### 8.1 RACH failure 분석 요청 프롬프트

```text
@HANDOFF.md
@rca_kg/schema/rca_case.schema.yaml
@rca_kg/schema/taxonomy.yaml
@rca_kg/indexes/index.md
@rca_kg/cases/EXAMPLE_v2_rach_failure_001.yaml
@rca_kg/cases/unresolved/EXAMPLE_unresolved.yaml
@rca_kg/skills_seed/rach_failure_analyzer.md
@rca_kg/signals/20260622_rach_failure_dump001_signal.txt

위 signal 파일을 schema v2 기준으로 분석해줘.

issue_type은 rach_failure 후보로 시작하되, 로그 근거가 다르면 이유를 설명해줘.
분석 기준은 rca_kg/skills_seed/rach_failure_analyzer.md 의 checklist를 따른다.

반드시 지켜야 할 규칙:
1. case_id는 날짜 기반으로 만들지 말고 fingerprint 기반으로 만들어줘.
   형식: <fingerprint-slug>_<issue_type>_<3-digit-seq>
2. fingerprint 블록을 반드시 생성해줘.
   - signature_set: 등장한 signature ID 목록
   - sequence: cptime 기준 상대 순서
   - sequence_status: 초안이면 draft
3. line_range를 사용하지 말고 cptime_range를 사용해줘.
4. raw_examples에 로그 원문을 인라인 저장하지 마.
5. related.jira를 만들지 말고, Jira가 있으면 recent_occurrences[].jira에 기록해줘.
6. 신규 case 생성 전 rca_kg/cases/ 아래 기존 case와 fingerprint(signature_set + sequence)를 비교해줘.
7. fingerprint가 기존 case와 일치하면 신규 YAML을 만들지 말고 기존 case의 occurrence_count, recent_occurrences, last_seen만 갱신해줘.
8. fingerprint가 일치하지 않으면 rca_kg/cases/<fingerprint-slug>_rach_failure_<seq>.yaml 로 신규 생성해줘.
9. 담당영역 밖 문제로 판단되면 rca_kg/cases/unresolved/<YYYYMMDD>_rach_failure_<seq>_PENDING.yaml 로 생성하고 root_cause/fix/review는 null로 둬.
10. crash/dump 분석은 5.5 RCA KG 대상이 아니므로 crash case를 생성하지 마.

저장 후 rca_kg/indexes/index.md를 fingerprint 기준으로 업데이트해줘.
변경 내용 요약을 답변에 포함해줘.
```

### 8.2 SCG failure 분석 요청 프롬프트

```text
@HANDOFF.md
@rca_kg/schema/rca_case.schema.yaml
@rca_kg/schema/taxonomy.yaml
@rca_kg/indexes/index.md
@rca_kg/cases/EXAMPLE_v2_rach_failure_001.yaml
@rca_kg/cases/unresolved/EXAMPLE_unresolved.yaml
@rca_kg/skills_seed/scg_failure_analyzer.md
@rca_kg/signals/20260622_scg_failure_dump001_signal.txt

위 signal 파일을 schema v2 기준으로 분석해줘.

issue_type은 scg_failure 후보로 시작하되, 로그 근거가 다르면 이유를 설명해줘.
분석 기준은 rca_kg/skills_seed/scg_failure_analyzer.md 의 checklist를 따른다.

반드시 지켜야 할 규칙:
1. case_id는 날짜 기반으로 만들지 말고 fingerprint 기반으로 만들어줘.
2. fingerprint.signature_set과 fingerprint.sequence를 반드시 생성해줘.
3. sequence는 cptime 기준 상대 순서로 작성해줘.
4. line_range, raw_examples, related.jira는 사용하지 마.
5. Jira 참조는 recent_occurrences[].jira에만 기록해줘.
6. 기존 cases/와 fingerprint 매칭 후 재발이면 occurrence_count만 갱신해줘.
7. 신규 원인이면 cases/에 신규 YAML을 생성해줘.
8. 담당영역 밖이면 cases/unresolved/에 PENDING YAML을 생성해줘.
9. crash/dump case는 생성하지 마.

저장 후 rca_kg/indexes/index.md를 fingerprint 기준으로 업데이트해줘.
변경 내용 요약을 답변에 포함해줘.
```

---

## 9. Step 5 — 생성된 case YAML 검토

Claude Code 또는 Roo Code가 생성한 YAML을 사람이 검토한다.

검토 파일 위치:

```text
rca_kg/cases/<fingerprint-slug>_<issue_type>_<seq>.yaml
```

또는 담당영역 밖 이관이면:

```text
rca_kg/cases/unresolved/<YYYYMMDD>_<issue_type>_<seq>_PENDING.yaml
```

### 9.1 필수 검토 항목

| 항목 | 확인 내용 |
|---|---|
| `issue_type` | taxonomy의 active issue_type 중 하나인지 |
| `fingerprint.signature_set` | 실제 signal에 등장한 signature만 포함했는지 |
| `fingerprint.sequence` | cptime 기준 상대 순서가 맞는지 |
| `fingerprint.sequence_status` | 자동 초안이면 draft, 사람이 확인했으면 confirmed인지 |
| `log_patterns[].signature` | signature ID가 fingerprint와 같은 네임스페이스인지 |
| `log_patterns[].cptime_range` | line_range가 아니라 cptime_range인지 |
| `root_cause.category` | taxonomy의 active root_cause_categories 중 하나인지 |
| `root_cause.confidence` | low / medium / high / confirmed 기준이 과하지 않은지 |
| `recent_occurrences[].jira` | Jira 참조가 이 위치에만 있는지 |
| `related` | hld / tc / api만 있는지 |
| `fix` | 확정 전이면 과하게 단정하지 않았는지 |
| `prevent_rule` | 실제 TC나 방어 규칙 후보로 쓸 수 있는지 |
| `review.status` | draft / reviewed / confirmed / rejected 중 하나인지 |

### 9.2 사용 금지 필드

schema v2 기준으로 아래 필드는 신규 case에 넣지 않는다.

```text
symptom.occurred_at
log_patterns.raw_examples
log_patterns.line_range
log_patterns.time_range
related.jira
issue_type: crash
```

---

## 10. Step 6 — review 상태 업데이트

초안 상태 예시:

```yaml
status: analyzed
fingerprint:
  sequence_status: draft
review:
  status: draft
  reviewer: ""
  reviewed_at: null
  comment: ""
```

사람이 검토한 뒤:

```yaml
status: reviewed
fingerprint:
  sequence_status: confirmed
review:
  status: reviewed
  reviewer: whpark
  reviewed_at: 2026-06-22
  comment: "fingerprint sequence 확인 완료"
```

Root Cause와 Fix까지 확정되면:

```yaml
status: confirmed
root_cause:
  confidence: confirmed
fix:
  confirmed_at: 2026-06-22
  confirmed_by: whpark
review:
  status: confirmed
```

---

## 11. Step 7 — unresolved 처리

`unresolved`는 원인 불명이 아니다.

```text
unresolved = 담당영역 밖으로 판단되어 타 담당자에게 이관해야 하는 상태
```

원인이 불확실하지만 담당영역 안이면 아래처럼 둔다.

```yaml
status: analyzed
root_cause:
  confidence: low
```

담당영역 밖이면 아래처럼 만든다.

```yaml
issue_type: scg_failure
status: unresolved
handoff:
  reason: "RF 캘리브레이션 영역 문제로 추정되어 본 담당 범위 밖"
  suspected_domain: "RF_Calibration"
  suspected_owner: "RF팀"
  handed_off_at: 2026-06-22
root_cause: null
fix: null
review: null
```

파일 위치:

```text
rca_kg/cases/unresolved/<YYYYMMDD>_<issue_type>_<seq>_PENDING.yaml
```

이관받은 담당자가 root_cause / fix / review를 채운 뒤에는 fingerprint 기준으로 기존 cases와 다시 매칭한다.

```text
fingerprint 일치: 기존 case에 병합하고 PENDING 파일 폐기
fingerprint 불일치: 정식 case_id 부여 후 cases/로 승격
```

---

## 12. Step 8 — index.md 업데이트

case 생성 또는 갱신 후 `rca_kg/indexes/index.md`를 업데이트한다.

index 기준 필드:

```text
case_id
issue_type
signature_set
sequence
occurrence_count
first_seen
last_seen
status
```

동일 fingerprint 재발이면 신규 행을 추가하지 않고 기존 행을 갱신한다.

```text
occurrence_count 증가
last_seen 갱신
status 필요 시 갱신
```

---

## 13. Step 9 — skills_seed 업데이트 후보

새로운 분석 패턴이 발견되면 바로 정식 반영하지 말고 후보로 기록한다.

예시 요청:

```text
이번 case에서 새로 확인된 RACH failure 분석 체크포인트를
rca_kg/skills_seed/rach_failure_analyzer.md 하단에 candidate로 추가해줘.
확정 키워드와 추정 키워드를 구분해서 표시해줘.
```

`keywords.yaml`이 signature SSOT다.
R4 이후에는 `keywords.yaml`을 기준으로 skills_seed, ps1 키워드, L1SW manifest fragment를 동기화한다.

---

## 14. 파일 역할 한눈에 보기

```text
RCA_standalone/
  ├─ HANDOFF.md
  │    └─ 새 세션 시작 시 가장 먼저 읽는 문서
  │
  ├─ scripts/
  │    ├─ README.md
  │    │    └─ 이 파일. schema v2 기준 운영 가이드
  │    ├─ rach_failure_prefilter.ps1
  │    │    └─ R3 keywords.yaml 전까지 사용하는 임시 RACH signal 추출 스크립트
  │    └─ scg_failure_prefilter.ps1
  │         └─ R3 keywords.yaml 전까지 사용하는 임시 SCG signal 추출 스크립트
  │
  └─ rca_kg/
       ├─ signals/
       │    └─ L1SW _l1sw.txt 또는 issue_type별 signal 파일 저장 위치
       │
       ├─ cases/
       │    ├─ EXAMPLE_v2_rach_failure_001.yaml
       │    │    └─ confirmed case 예시
       │    ├─ <fingerprint-slug>_<issue_type>_<seq>.yaml
       │    │    └─ 정식 RCA case
       │    └─ unresolved/
       │         └─ <YYYYMMDD>_<issue_type>_<seq>_PENDING.yaml
       │              └─ 담당영역 밖 이관 대기 case
       │
       ├─ skills_seed/
       │    ├─ rach_failure_analyzer.md
       │    ├─ scg_failure_analyzer.md
       │    ├─ tx_abnormal_analyzer.md
       │    ├─ l2_max_retransmission_analyzer.md
       │    └─ crash_analyzer.md
       │         └─ deprecated. 신규 5.5 case 생성에 사용하지 않음
       │
       ├─ indexes/
       │    └─ index.md
       │         └─ fingerprint 기준 case 검색 index
       │
       └─ schema/
            ├─ rca_case.schema.yaml
            │    └─ case YAML 필드 정의
            └─ taxonomy.yaml
                 └─ active issue_type / root_cause / confidence 기준
```

---

## 15. 문제 발생 시

| 증상 | 가능한 원인 | 조치 |
|---|---|---|
| signal 파일이 0줄 | 키워드 후보가 실제 로그와 맞지 않음 | `_l1sw.txt`에서 실제 표현을 검색하고 ps1 키워드 후보를 보정 |
| signal 파일이 너무 큼 | context가 너무 넓음 | `-Context 5`로 줄이거나 L1SW time window 사용 |
| Claude가 날짜 기반 case_id를 만들려고 함 | v1 가이드를 참조했거나 프롬프트가 불명확함 | schema v2와 EXAMPLE_v2 파일을 함께 첨부하고 fingerprint 기반 case_id를 재지시 |
| Claude가 line_range를 사용함 | v1 잔재 또는 일반 로그 분석 습관 | cptime_range만 사용하라고 재지시 |
| Claude가 raw_examples를 넣음 | 로그 원문 보존 원칙 미반영 | raw_examples 제거, signal_file + cptime_range 포인터로 대체 |
| Claude가 related.jira를 만듦 | v1 스키마 잔재 | Jira는 recent_occurrences[].jira로 이동 |
| crash case를 만들려고 함 | deprecated 파일 또는 과거 delta 참조 | crash는 L1SW 전담이며 5.5 active issue_type이 아니라고 재지시 |
| fingerprint가 과하게 길어짐 | noise signature가 포함됨 | 핵심 signature만 남기고 sequence_status를 draft로 유지한 뒤 사람이 정규화 |

---

## 16. 다음 단계

이 README는 schema v2 기준으로 동기화된 운영 가이드다.

다음 단계는 R4다.

```text
R4. L1SW manifest fragment 후보 설계
```

R4에서 정리할 내용:

```text
keywords.yaml에서 use_for.l1sw_manifest_fragment=true인 signature 추출
issue_type별 fragment JSON 후보 생성
너무 넓은 context keyword 제외
사내 L1SW manifest 구조와 충돌 여부 확인
실 로그 검증 전까지 fragment는 candidate로 유지
```
