# 5.5 RCA — Claude Code 자율 실행 프롬프트 모음 (E2E v1)

> 목적: 사람 작업을 최소화한다. 부족한 정보는 사람이 조사하지 않고,
> **사내에서 동작 중인 Claude Code에게 직접 환경을 조사(probe)시켜** 뽑아낸다.
> 사용처: 사내 PC, `RCA_standalone` 폴더를 연 VSCode + Claude Code.
>
> 사용법: 아래 프롬프트를 **위에서 아래 순서대로** Claude Code에 붙여넣는다.
> 각 프롬프트는 독립 실행 가능하며, 앞 단계 산출물을 다음 단계가 읽는다.
>
> 사람이 반드시 손대야 하는 지점은 각 프롬프트 끝의 `[사람 확인]` 으로 명시했다.
> 그 외에는 Claude Code가 조사·생성·기록까지 자동 수행한다.

---

## P0 — 환경 자가 진단 (가장 먼저 1회)

> Claude Code가 자기 환경을 스스로 조사해 "지금 무엇이 가능하고 무엇이 빠졌는지"를
> 보고한다. 사람은 보고서만 읽으면 된다.

```text
@HANDOFF.md
@scripts/README.md
@rca_kg/keywords.yaml
@rca_kg/schema/rca_case.schema.yaml

너는 지금 사내 RCA_standalone 작업 폴더 안에서 동작하고 있다.
아래를 직접 조사해서 "환경 진단 보고서"를 만들어줘. 추측하지 말고 실제로 확인해줘.

조사 항목:
1. 이 폴더에서 .sdm 또는 _l1sw.txt 파일이 있는지 찾아줘 (있으면 경로와 크기).
   - 없으면 "입력 로그 없음 → P1에서 L1SW 실행 필요" 로 표시.
2. L1SW Log Analyzer 스킬이 이 환경에서 접근 가능한지 확인해줘.
   - parse.ps1, manifest 디렉토리, fragment JSON 파일 위치를 찾아줘.
   - 찾으면 manifest 디렉토리 경로와 fragment JSON 파일 목록을 보고해줘.
   - 못 찾으면 어떤 경로를 뒤졌는지 알려줘.
3. PowerShell 실행이 가능한지 확인해줘 (rach_failure_prefilter.ps1 의 -InputLog/-OutputTxt/-Context 파라미터 구조 확인만, 실행은 하지 마).
4. rca_kg/cases/ 아래에 EXAMPLE 외 실제 case 파일이 있는지 세어줘.
5. rca_kg/signals/ 가 비어 있는지 확인해줘.

보고 형식 (표로):
| 항목 | 상태 (OK / 없음 / 확인불가) | 근거 경로 | 다음 조치 |

마지막에 "지금 P1부터 시작 가능한가?" 를 yes/no 로 판정하고,
no면 사람이 먼저 해줘야 할 최소 작업만 1~3줄로 알려줘.

파일은 아무것도 수정하지 마. 조사·보고만 해.
```

`[사람 확인]` 보고서의 "다음 조치"만 확인. L1SW 경로가 안 잡히면 그 경로만 알려주면 됨.

---

## P1 — L1SW 출력 형식 역설계 (manifest 후보의 전제)

> manifest fragment를 잘 만들려면 L1SW가 실제로 뱉는 `_l1sw.txt`의 줄 형식
> (cptime 표기, module 표기, 한 줄에 무엇이 들어가는지)을 알아야 한다.
> 이걸 사람이 읽지 않고 Claude Code가 샘플을 떠서 형식을 정리하게 한다.

```text
@scripts/README.md

목표: L1SW가 생성한 _l1sw.txt 의 실제 줄 형식을 역설계해줘.

지시:
1. 이 환경에서 _l1sw.txt 를 하나 찾아줘 (P0에서 찾았다면 그 경로 사용).
   - 없으면, .sdm 을 찾아 L1SW parse.ps1 로 _l1sw.txt 를 생성하는
     정확한 명령어만 제시하고 멈춰줘 (실행 권한이 있으면 실행해도 됨).
2. _l1sw.txt 앞부분 50줄과, 파일 전체에서 무작위 구간 30줄을 떠서 보여줘.
   (단, 사내 민감정보로 보이는 토큰은 <MASK> 로 가려줘.)
3. 아래를 형식 명세로 정리해줘:
   - cptime 이 줄 어디에 어떤 포맷으로 찍히는가 (예: [00123.456], 0x..., HH:MM:SS.mmm)
   - module/component 이름이 줄 어디에 찍히는가
   - severity / log level 표기가 있는가
   - UE / session / correlation id 로 쓸 수 있는 필드가 있는가
   - 한 줄이 대략 어떤 구조인가 (필드 순서)
4. 위 형식 명세를 prompt/deepdive/L1SW_OUTPUT_FORMAT_PROBED_<YYYYMMDD>_KST.md 로 저장해줘.
   파일 맨 위에 "이 문서는 실제 _l1sw.txt 샘플에서 역설계한 형식 명세다" 라고 명시하고,
   추정인 부분은 "추정", 샘플로 확인된 부분은 "확인됨" 으로 구분해줘.

이 결과는 P2(manifest fragment 후보)와 P4(분석)에서 cptime_range 추출 기준으로 쓰인다.
```

`[사람 확인]` 형식 명세에서 cptime 포맷만 맞는지 눈으로 1회 확인.

---

## P2 — L1SW manifest fragment 후보 자동 생성 (R4)

> keywords.yaml 에서 `use_for.l1sw_manifest_fragment: true` 인 signature만 뽑아
> issue_type별 fragment JSON 후보를 만든다. 사람이 키워드를 손으로 고르지 않는다.

```text
@rca_kg/keywords.yaml
@scripts/README.md
@prompt/deepdive/L1SW_OUTPUT_FORMAT_PROBED_<YYYYMMDD>_KST.md   # P1 산출물, 있으면 첨부

목표: L1SW manifest fragment 후보 JSON을 issue_type별로 자동 생성해줘 (R4).

규칙:
1. keywords.yaml 을 단일 출처로 사용해줘. 키워드를 새로 지어내지 마.
2. 각 issue_type 에서 use_for.l1sw_manifest_fragment == true 인 signature 만 포함해줘.
   - use_for.fingerprint == false 인 generic context signature(RACH/SCG/RLC/HARQ 등)는
     manifest fragment에서도 제외해줘 (noise_risk: high).
3. fragment JSON 구조는 사내 L1SW manifest 의 기존 fragment JSON 형식을 그대로 따라야 한다.
   - P0에서 찾은 기존 fragment JSON(예: 모듈 매핑용)을 1개 열어서 키 구조를 확인하고,
     동일한 키 구조로 만들어줘. (L1SW의 Select-String 이 줄 전체 매칭이라는 전제는 유지)
   - 기존 fragment 구조를 못 찾으면, keywords.yaml 의 signature ID / regex / aliases 를
     담은 잠정 구조로 만들고 "구조 미확인 — 사내 fragment와 대조 필요" 라고 명시해줘.
4. 출력 파일:
   - rca_kg/manifest_fragments/rca_rach.json
   - rca_kg/manifest_fragments/rca_scg.json
   - rca_kg/manifest_fragments/rca_tx.json
   - rca_kg/manifest_fragments/rca_l2.json
   각 JSON에 source: "keywords.yaml v0.1", status별(confirmed/candidate) 구분을 주석 또는
   필드로 남겨줘.
5. 각 signature 의 status(confirmed/candidate)를 fragment 안에 보존해줘.
   confirmed 와 candidate 를 한눈에 구분할 수 있어야 한다.
6. 생성 후 review_logs/rca_standalone_R4_manifest_fragments_<YYYYMMDD>_<HHMM>_KST.md 에
   - 어떤 signature를 포함/제외했는지
   - 제외 사유(generic/noise)
   - 기존 L1SW fragment 구조와 일치 여부
   를 표로 기록해줘.

keywords.yaml 자체는 수정하지 마. fragment는 candidate 산출물이다.
```

`[사람 확인]` 생성된 fragment JSON을 사내 L1SW manifest 디렉토리에 넣을지 여부만 결정.
넣기 전 review_log의 "기존 구조 일치 여부" 표만 확인.

---

## P3 — signal 생성 (입력 만들기)

> 두 경로 중 환경에 맞는 쪽을 Claude Code가 알아서 고른다.

```text
@scripts/README.md
@rca_kg/keywords.yaml

목표: 분석할 issue_type 의 signal 파일을 만들어줘.
이번 대상 issue_type: <rach_failure | scg_failure | tx_abnormal | l2_max_retransmission>
입력 _l1sw.txt 경로: <P0/P1에서 확인된 경로>

진행:
1. 입력 _l1sw.txt 가 작으면(예: 5만 줄 이하) pre-filter 없이 그대로 signal 로 써도 된다.
   그 경우 rca_kg/signals/<YYYYMMDD>_<issue_type>_<source-slug>_signal.txt 로 복사해줘.
2. 크면 pre-filter 가 필요하다. 이때 키워드는 keywords.yaml 에서
   해당 issue_type 의 use_for.prefilter == true 인 signature의 regex/aliases 를 사용해줘.
   - PowerShell 실행이 가능하면 scripts/<issue_type>_prefilter.ps1 를 사용하되,
     스크립트 내부 키워드가 keywords.yaml 과 다르면 차이를 먼저 보고해줘 (자동 수정 금지).
   - PowerShell 을 못 쓰면, keywords.yaml 기반으로 동등한 grep/Select-String 명령을
     만들어 직접 signal 을 추출해줘.
3. 생성된 signal 파일에 대해:
   - 줄 수
   - 첫 30줄 미리보기
   - 매칭된 signature ID별 등장 횟수 (keywords.yaml ID 기준 집계)
   를 보고해줘.
4. signal 줄 수 판단 기준(scripts/README.md 7장)에 따라
   너무 적음/적정/너무 많음 을 판정하고, 필요하면 -Context 조정안을 제시해줘.

signal 파일만 만들고, case YAML 은 아직 만들지 마 (P4에서 한다).
```

`[사람 확인]` signature별 등장 횟수만 훑어보면 "이 로그에 진짜 그 증상이 있나" 가 보임. 없으면 issue_type 잘못 고른 것.

---

## P4 — RCA 분석 + case YAML 생성 (핵심 루프)

> scripts/README 8장 프롬프트를 자율형으로 강화. issue_type을 한 줄만 바꾸면 4종 모두 동작.

```text
@HANDOFF.md
@rca_kg/schema/rca_case.schema.yaml
@rca_kg/schema/taxonomy.yaml
@rca_kg/keywords.yaml
@rca_kg/indexes/index.md
@rca_kg/cases/EXAMPLE_v2_rach_failure_001.yaml
@rca_kg/cases/unresolved/EXAMPLE_unresolved.yaml
@rca_kg/skills_seed/<issue_type>_analyzer.md
@rca_kg/signals/<P3에서 만든 signal 파일>

위 signal 을 schema v2 기준으로 분석해서 RCA case 를 생성해줘.
이번 issue_type 후보: <issue_type>  (로그 근거가 다르면 다른 issue_type 으로 바꾸고 이유를 설명)

반드시 지킬 규칙 (schema v2):
1. case_id = <fingerprint-slug>_<issue_type>_<3-digit-seq>  (날짜 기반 금지).
2. fingerprint 블록 필수:
   - signature_set: signal 에 실제 등장한 signature ID (keywords.yaml ID 만 사용).
   - sequence: cptime 기준 상대 순서 (절대시각 미포함).
   - sequence_status: 자동 추출이므로 draft.
   - generic(use_for.fingerprint=false) signature 는 signature_set 에 넣지 마.
3. 위치 표기는 cptime_range 만 사용 (line_range / time_range / raw_examples 금지).
4. Jira 는 recent_occurrences[].jira 에만. related 에는 hld/tc/api 만.
5. 신규 case 생성 전 rca_kg/cases/ 의 기존 case 들과 fingerprint(signature_set+sequence)를 비교해줘.
   - 일치: 신규 YAML 만들지 말고 기존 case 의 occurrence_count++,
           recent_occurrences 추가, last_seen 갱신만.
   - 불일치: rca_kg/cases/<fingerprint-slug>_<issue_type>_<seq>.yaml 신규 생성.
6. 담당영역 밖이면 rca_kg/cases/unresolved/<YYYYMMDD>_<issue_type>_<seq>_PENDING.yaml,
   root_cause/fix/review = null, handoff 블록 필수.
   (원인 불확실이지만 담당영역 안이면 status:analyzed + confidence:low 로, unresolved 아님)
7. crash/dump case 는 생성 금지 (L1SW 전담).
8. signature_set 에 쓴 signature 중 keywords.yaml 에서 status:candidate 인 것이 있으면,
   "이 case 의 실제 로그에서 그 표현이 확인됐다" 는 근거(해당 로그 줄 인용 1줄)를 보고해줘.
   → 이 근거는 나중에 candidate→confirmed 승격 판단에 쓴다.

저장 후:
- rca_kg/indexes/index.md 를 fingerprint 기준으로 갱신.
- 답변에 변경 요약 + signature_set + sequence + 재발/신규/이관 판정 결과를 표로 정리.
- candidate signature 중 이번에 실로그로 확인된 ID 목록을 따로 알려줘 (P6 입력).
```

`[사람 확인]` 변경 요약 표 1개만 확인. review.status 승인은 P5에서 일괄.

---

## P5 — 검토 일괄 승인 (사람 작업의 거의 전부)

> 사람이 손대는 거의 유일한 실질 단계. 그마저도 Claude Code가 체크리스트를 채워주고
> 사람은 OK/수정만 한다.

```text
@scripts/README.md
@rca_kg/schema/rca_case.schema.yaml
@rca_kg/cases/<P4에서 생성/갱신된 case 파일>

위 case YAML 을 scripts/README.md 9.1 의 필수 검토 항목 표 기준으로 자가 점검해줘.
각 항목을 OK / 의심 / 위반 으로 판정하고, 의심·위반은 근거 줄을 보여줘.

특히:
- fingerprint.signature_set 에 generic signature 가 섞이지 않았는지
- cptime_range 만 썼는지 (line_range/time_range/raw_examples 금지 위반 없는지)
- root_cause.category 가 taxonomy active 목록 안인지
- issue_type 이 crash 가 아닌지

자가 점검 후, 사람이 승인하면 적용할 'review 승인 패치'를 미리 만들어서 보여줘
(아직 적용하지 마):
  status: reviewed
  fingerprint.sequence_status: confirmed
  review.status: reviewed
  review.reviewer: whpark
  review.reviewed_at: <오늘>
  review.comment: "<핵심 한 줄>"

내가 "승인" 이라고 하면 그 패치를 case 파일에 적용하고 index.md 의 status 도 갱신해줘.
내가 특정 필드를 지적하면 그 부분만 고치고 다시 점검표를 보여줘.
```

`[사람 확인]` 점검표 보고 **"승인"** 한 단어 입력. 끝.

---

## P6 — keywords.yaml 승격 (candidate → confirmed, 누적 자산화)

> 실로그로 확인된 candidate signature를 confirmed로 올린다. SSOT가 검증과 함께 자라게 하는 단계.

```text
@rca_kg/keywords.yaml
@rca_kg/schema/keywords.schema.yaml
@rca_kg/cases/<이번에 만든 case 파일>

이번 분석(P4)에서 "실제 로그 줄로 확인됨" 으로 보고된 candidate signature 들이 있다.
대상 signature ID 목록: <P4가 알려준 목록>

작업:
1. 각 대상 signature 에 대해, case 파일과 signal 의 실제 로그 줄을 근거로
   confirmed 승격이 타당한지 다시 판정해줘 (근거 줄 인용).
2. 타당한 것만 keywords.yaml 에서 status: candidate → confirmed 로 바꾸고,
   note 에 "실로그 확인: <case_id>, cptime <범위>" 를 덧붙여줘.
3. 애매한 것은 candidate 로 두고 note 에만 "1회 관측, 추가 확인 필요" 를 남겨줘.
4. 변경 사항을 review_logs/rca_standalone_keywords_promote_<YYYYMMDD>_<HHMM>_KST.md 에
   before/after 표로 기록해줘.
5. keywords.yaml 의 version 을 0.1 → 0.2 로 올리고 metadata.note 를 갱신해줘.

generic_keyword_policy 는 유지해줘 — RACH/SCG/RLC/HARQ 는 confirmed 여도
use_for.fingerprint 는 false 그대로 둬.
```

`[사람 확인]` before/after 표만 확인. 승격이 과하면 해당 ID만 되돌리라고 지시.

---

## 부록 — 한 줄 요약 흐름

```text
P0 진단 → P1 형식역설계 → P2 manifest후보(R4) → P3 signal → P4 분석/case → P5 승인 → P6 SSOT승격
          └ 사람: 보고서만   └ 사람: cptime만   └ 사람: 구조일치만  └ 사람: 횟수만  └ 사람: "승인"  └ 사람: 표만
```

사람의 실질 입력은 P5의 "승인" 한 번과, 각 단계 보고서 훑어보기뿐이다.
나머지 조사·생성·기록은 Claude Code가 환경을 직접 probe 해서 수행한다.
```
