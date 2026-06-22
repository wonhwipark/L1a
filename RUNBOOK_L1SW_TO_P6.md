# 5.5 RCA — 수행 순서 설명서 (L1SW 연동 → P0~P6)

작성일: 2026-06-23 KST
대상: 사내 PC, `RCA_standalone` 폴더를 연 VSCode + Claude Code
목적: **L1SW 스킬 연동부터** E2E 프롬프트(P0~P6)까지, 처음 하는 사람도 순서대로 따라
      하면 되도록 한 줄로 꿴 실행 설명서.

> 이 문서는 "무엇을 왜"는 `USAGE_SCENARIO.md`, "프롬프트 본문"은
> `prompt/CLAUDE_CODE_PROMPTS_e2e_v1.md`, "원인분석 추론법"은
> `prompt/deepdive/RCA_ANALYSIS_METHODOLOGY.md` 에 있다. 여기서는 **순서와 손동작**만 다룬다.

---

## 0. 전체 순서 한 장

```
[준비]  STEP A  L1SW 스킬 연동 확인 (skills 경로 · parse.ps1 · manifest)
          │
[진단]  STEP B  P0  환경 자가 진단 ─────────────── 사람: 보고서 확인
          │
[형식]  STEP C  P1  _l1sw.txt 형식 역설계 ───────── 사람: cptime 포맷 확인
          │
[준비2] STEP D  P2  manifest fragment 후보 생성(R4) ─ 사람: 구조 일치 확인
          │
        ───── 여기까지 1회 셋업. 이후 로그마다 STEP E~I 반복 ─────
          │
[입력]  STEP E  L1SW 실행 → _l1sw.txt 생성 (사람이 실행)
          │
[추출]  STEP F  P3  signal 생성 ────────────────── 사람: signature 횟수 확인
          │
[분석]  STEP G  P4  원인분석(7단계) + case YAML ── 사람: 변경 요약 확인
          │
[승인]  STEP H  P5  자가점검 → "승인" ──────────── 사람: "승인" 한 단어
          │
[자산화] STEP I  P6  keywords 승격(candidate→confirmed) 사람: 표 확인
```

사람의 실질 입력: STEP E(L1SW 실행) + STEP H("승인") 두 번. 나머지는 보고서 확인뿐.

---

## STEP A — L1SW 스킬 연동 확인 (최초 1회)

P0를 돌리기 전에 **L1SW Log Analyzer 스킬이 이 PC에서 동작하는지**부터 확인한다.
P0가 자동으로 찾아주긴 하지만, 경로를 미리 알면 P0 보고가 훨씬 정확해진다.

### A-1. 스킬 위치 확인
L1SW 스킬은 보통 아래 경로에 있다(메모리 기준, 환경마다 다를 수 있음):

```text
C:\Users\whpark\.claude\skills\l1sw-log-analyzer\
   └ scripts\parse.ps1            ← 핵심 실행 스크립트
   └ scripts\sdm_to_test.ps1      ← parse.ps1 이 내부 호출 (.sdm → 평문)
   └ manifest\<part>.json          ← 모듈명→regex 매핑 SSOT (5개 fragment)
```

PowerShell에서:
```powershell
Test-Path "C:\Users\whpark\.claude\skills\l1sw-log-analyzer\scripts\parse.ps1"
Get-ChildItem "C:\Users\whpark\.claude\skills\l1sw-log-analyzer\manifest\"
```
- `True` 와 fragment JSON 목록이 보이면 연동 OK.
- 경로가 다르면 실제 경로를 적어둔다(STEP B P0 보고에 사용).

### A-2. RCA_standalone 가 작업 루트인지 확인
Claude Code/VSCode에서 **`RCA_standalone` 폴더 자체를 연다**. 상위 폴더를 열면
프롬프트의 `@상대경로` 참조가 깨진다(HANDOFF 명시).

### A-3. (선택) MCP 영속 설정
Jira/Confluence를 쓸 계획이면 `prompt/MCP_PERSIST_SETUP_PROMPTS.md` 의 A 프롬프트로
`.mcp.json` 을 먼저 깔아두면 새 세션마다 끊기지 않는다. RCA 분석 자체에는 필수 아님.

> 이 단계 산출물은 없다. "parse.ps1 경로 + manifest 디렉토리 경로"를 손에 쥐는 게 전부.

---

## STEP B — P0 환경 자가 진단

`prompt/CLAUDE_CODE_PROMPTS_e2e_v1.md` 의 **P0** 블록을 통째로 복사해 Claude Code에 붙여넣는다.

- **무엇을 하나**: Claude Code가 `.sdm`/`_l1sw.txt` 존재, L1SW 경로, PowerShell 가능 여부,
  기존 case 수, signals 비었는지를 스스로 조사해 표로 보고한다.
- **사람이 보는 것**: 표의 "다음 조치" 열. 맨 아래 "지금 P1부터 시작 가능한가? yes/no".
- **막히면**: no 가 나오면 STEP A에서 적어둔 L1SW 경로만 알려주면 된다.

✋ 사람 입력: 보고서 확인. (파일 수정 안 함 — 조사 전용)

---

## STEP C — P1 `_l1sw.txt` 형식 역설계

**P1** 블록을 붙여넣는다.

- **무엇을 하나**: 실제 `_l1sw.txt` 샘플을 떠서 cptime 표기·module 표기·줄 구조를 역설계하고
  `prompt/deepdive/L1SW_OUTPUT_FORMAT_PROBED_<날짜>_KST.md` 로 저장한다.
- **왜 먼저**: 이 형식을 알아야 P2의 manifest fragment와 P4의 cptime_range 추출이 정확해진다.
- **사람이 보는 것**: cptime 포맷 한 줄(예 `[00123.456]`)이 실제와 맞는지 눈으로 1회.

✋ 사람 입력: cptime 포맷만 확인.

> `_l1sw.txt` 가 아직 없으면 P1이 "parse.ps1로 생성하는 명령"을 제시하고 멈춘다.
> 그 경우 STEP E를 먼저 한 번 돌린 뒤 P1로 돌아온다.

---

## STEP D — P2 manifest fragment 후보 생성 (R4)

**P2** 블록을 붙여넣는다.

- **무엇을 하나**: `keywords.yaml` 에서 `use_for.l1sw_manifest_fragment: true` 인 signature만
  뽑아 issue_type별 fragment JSON 후보(`rca_rach.json` 등)를 `rca_kg/manifest_fragments/` 에
  생성한다. generic(RACH/SCG/RLC/HARQ)은 제외. review_log에 포함/제외 근거를 표로 남긴다.
- **사람이 보는 것**: review_log의 "기존 L1SW fragment 구조 일치 여부" 표.
- **그다음**: 일치하면 이 JSON을 사내 L1SW manifest 디렉토리에 넣어 1차 필터 자체를
  issue_type 인지형으로 만들 수 있다(선택).

✋ 사람 입력: 구조 일치만 확인. 사내 manifest에 반영할지 결정.

> 여기까지가 **최초 1회 셋업**. STEP B~D는 환경이 바뀌지 않는 한 다시 안 해도 된다.

---

## STEP E — L1SW 실행 (로그마다 반복 시작)

분석할 `.sdm` 로그가 생기면 L1SW로 1차 축소본을 만든다. (사람이 실행)

```powershell
cd "C:\Users\whpark\.claude\skills\l1sw-log-analyzer\scripts"
# parse.ps1 의 정확한 입력 파라미터명은 환경마다 다를 수 있다.
# STEP B(P0)/STEP C(P1) 이 실제 호출 형태를 조사해 알려준다 — 그 형태를 그대로 쓴다.
.\parse.ps1 <P0/P1이 알려준 입력 파라미터> "D:\work\logs\dump001.sdm"
# 옵션: -Modules <부분집합>  -TimeFrom/-TimeTo <시간창>
# stdout 마지막 줄에 생성된 _l1sw.txt 경로가 찍힌다 — 그 경로를 복사해 둔다.
```

- 산출: `_l1sw.txt` (예: 704MB → 5MB, 약 1/140 축소).
- 이 파일은 **삭제하지 않고 보존**한다. case는 본문을 복사하지 않고 이 경로 + cptime만 가리킨다.
- crash/dump 분석이 목적이면 여기서 L1SW가 전담한다. 5.5는 crash case를 만들지 않는다.

✋ 사람 입력: parse.ps1 실행, 출력된 `_l1sw.txt` 경로 확보.

---

## STEP F — P3 signal 생성

**P3** 블록을 붙여넣되, 상단 3개 자리표시자를 채운다:
- 대상 issue_type: `rach_failure | scg_failure | tx_abnormal | l2_max_retransmission` 중 하나
- 입력 `_l1sw.txt` 경로: STEP E에서 복사한 경로

- **무엇을 하나**: `_l1sw.txt` 가 작으면 그대로 signal로 복사, 크면 `keywords.yaml` 기반
  pre-filter로 `rca_kg/signals/<날짜>_<issue_type>_<source>_signal.txt` 생성.
  PowerShell 가능하면 `scripts/<issue_type>_prefilter.ps1` 사용(키워드가 keywords.yaml과
  다르면 차이를 먼저 보고하고 자동수정 안 함).
- **사람이 보는 것**: signature ID별 등장 횟수. **여기서 0이면 issue_type을 잘못 고른 것** —
  횟수가 잡히는 issue_type으로 바꿔 다시 돌린다.

✋ 사람 입력: signature 등장 횟수만 훑기.

---

## STEP G — P4 원인분석(7단계) + case YAML 생성 ★핵심

**P4** 블록을 붙여넣고 issue_type 자리표시자와 P3 signal 파일 경로를 채운다.

- **PART A (원인분석)**: `RCA_ANALYSIS_METHODOLOGY.md` 7단계를 수행한다.
  증상 anchor 고정 → 시간창 → 정상경로 대조로 최초 이탈 특정 → 가설 2~4개 →
  가르는 단서만 grep → 단말/환경 분리 → 인과사슬 + confidence.
  각 단계 결론과 근거 cptime이 답변에 표시된다.
- **PART B (case YAML)**: PART A 결과를 schema v2 필드에 매핑.
  - `fingerprint.signature_set/sequence` = keywords.yaml **ID 표기**(alias 금지).
  - `root_cause.summary` = 인과사슬 문장(증상 나열 금지).
  - `root_cause.confidence` = 증거 충족도(§3 표)대로.
  - 위치는 `cptime_range` 만(line/time/raw 금지).
  - 기존 case와 fingerprint 비교 → 일치면 `occurrence_count++`만, 불일치면 신규.
  - 환경/RF 원인이면 `cases/unresolved/...PENDING.yaml` + handoff(`root_cause=null`).
- **사람이 보는 것**: 변경 요약 표 + 인과사슬 1줄 + confidence 근거 + 재발/신규/이관 판정.

✋ 사람 입력: 변경 요약 표 1개 확인. (승인은 STEP H)

---

## STEP H — P5 자가점검 → "승인"

**P5** 블록을 붙여넣고 P4가 만든/갱신한 case 파일 경로를 채운다.

- **무엇을 하나**: case를 필수 검토 항목으로 자가점검(OK/의심/위반)하고, 특히
  ID 표기 위반 / 인과사슬 vs 증상나열 / confidence 정합 / 환경원인을 low로 들고 있지 않은지
  점검한다. 통과하면 적용할 'review 승인 패치'를 미리 보여준다(아직 적용 안 함).
- **사람이 보는 것**: 점검표.
- **승인**: 문제 없으면 **`승인`** 한 단어를 입력한다. 그러면 case에 `status: reviewed`,
  `review.reviewer: whpark` 등이 적용되고 index.md도 갱신된다.
  특정 필드가 이상하면 그 필드만 지적 → 그 부분만 고치고 점검표를 다시 보여준다.

✋ 사람 입력: **"승인"** (또는 고칠 필드 지적).

---

## STEP I — P6 keywords.yaml 승격 (자산 누적)

**P6** 블록을 붙여넣고 P4가 알려준 "실로그로 확인된 candidate signature ID 목록"을 채운다.

- **무엇을 하나**: 실제 로그 줄로 확인된 candidate signature를 `keywords.yaml` 에서
  `candidate → confirmed` 로 올리고 note에 근거(case_id, cptime)를 남긴다.
  버전을 0.1→0.2로 올린다. generic_keyword_policy 유지(RACH/SCG/RLC/HARQ는 confirmed여도
  fingerprint 미사용).
- **사람이 보는 것**: before/after 표. 승격이 과하면 해당 ID만 되돌리라고 지시.
- **의미**: 이 단계가 **닫힌 학습 루프**다. 분석할수록 SSOT가 검증과 함께 자란다.

✋ 사람 입력: before/after 표 확인.

---

## 반복 구조 정리

```text
최초 1회 :  STEP A(L1SW 연동) → B(P0) → C(P1) → D(P2)
로그마다 :  STEP E(L1SW 실행) → F(P3) → G(P4) → H(P5) → I(P6)
              └ 같은 fingerprint 재발이면 G에서 occurrence_count++ 로 끝(case 미증식)
              └ 담당영역 밖이면 G에서 unresolved 로 분기, 이관 후 담당자가 채워 통합
```

## 막혔을 때 빠른 분기

| 증상 | 원인 | 조치 |
|---|---|---|
| P0가 L1SW 못 찾음 | 스킬 경로 다름 | STEP A-1 경로를 P0에 알려줌 |
| P1이 멈추고 명령만 줌 | `_l1sw.txt` 없음 | STEP E 먼저 1회 |
| P3 signature 횟수 0 | issue_type 오선택 | 횟수 잡히는 issue_type으로 P3 재실행 |
| P4 confidence 애매 | 최초 이탈 위 단서 부족 | `analyzed`+`low` 유지(담당영역 안) |
| 원인이 RF/환경 | 담당영역 밖 | `unresolved`+handoff (low로 우기지 말 것) |
| 모바일/외부 PC | L1SW·PowerShell 불가 | STEP E~F는 사내 PC 전용 |

---

## 한 줄 요약

```
L1SW 연동 확인(A) → P0 진단 → P1 형식 → P2 fragment → [로그] L1SW 실행 →
P3 signal → P4 원인분석+case → P5 "승인" → P6 승격.
사람은 L1SW 실행과 "승인" 두 번만. 분석 품질은 METHODOLOGY 7단계가 책임진다.
```
