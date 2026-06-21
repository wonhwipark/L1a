# 5.5 RCA Knowledge Graph — 전체 사용 시나리오

> 이 문서는 "그래서 실제로 어떻게 돌리는가"를 처음부터 끝까지 한 장에 보여준다.
> 설계 의도·결정 이력은 `HANDOFF.md` 와 `prompt/deepdive/` 를 본다.
> 단계별 실행 프롬프트 전문은 `prompt/CLAUDE_CODE_PROMPTS_e2e_v1.md` 를 본다.
> 수동 절차 상세는 `scripts/README.md` 를 본다.
>
> 최종 업데이트: 2026-06-22 0805 KST

---

## 0. 한눈에 보는 전체 흐름

```text
[사내 PC / VSCode + Claude Code, RCA_standalone 폴더]

  .sdm 원본 로그
      │
      ▼  ① L1SW Log Analyzer (parse.ps1)        ← 1차 축소 ~80%, crash 전담
  _l1sw.txt
      │
      ▼  ② (선택) issue_type pre-filter           ← 2차, 증상 키워드 grep
  signal_<issue_type>.txt
      │
      ▼  ③ Claude Code 분석 (schema v2)
      │     - issue_type 분류
      │     - fingerprint 생성 (signature_set + cptime sequence)
      │     - 기존 cases/ 와 fingerprint 매칭
      ▼
  ┌─ 재발 일치 → 기존 case occurrence_count++ + recent_occurrences
  ├─ 신규      → cases/<fingerprint-slug>_<issue_type>_<seq>.yaml
  └─ 담당 밖   → cases/unresolved/<date>_<type>_<seq>_PENDING.yaml
      │
      ▼  ④ 사람 검토 → "승인" (review.status, sequence_status 정규화)
      │
      ▼  ⑤ index.md 갱신 + keywords.yaml candidate→confirmed 승격
  누적 Knowledge Graph 성장
```

**역할 분담 한 줄 정리**

| 주체 | 하는 일 |
|---|---|
| L1SW Log Analyzer | `.sdm` → `_l1sw.txt` 1차 축소, HTML 리포트, **crash/dump 전담** |
| 5.5 pre-filter (ps1) | `_l1sw.txt` → issue_type 특화 2차 필터 (증상/프로토콜 키워드) |
| Claude Code | 유형분류 · fingerprint · case YAML 생성 · index/skills_seed 갱신 |
| 사람 | L1SW 실행, 보고서 확인, **"승인" 한 번**, 외부자산(Jira/HLD/TC) 연결 |

---

## 1. 두 가지 운영 모드

### 모드 A — 자율형 (권장, 사람 작업 최소)

`prompt/CLAUDE_CODE_PROMPTS_e2e_v1.md` 의 P0~P6 프롬프트를 순서대로 사용한다.
Claude Code 가 환경을 직접 조사(probe)해서 빈칸을 채운다.
사람은 보고서를 훑고 P5에서 "승인" 만 입력한다.

```text
P0 진단 → P1 형식역설계 → P2 manifest후보 → P3 signal → P4 분석/case → P5 승인 → P6 SSOT승격
```

### 모드 B — 수동형 (단계별 통제)

`scripts/README.md` 의 Step 1~9 절차를 사람이 직접 따라간다.
PowerShell 명령, signal 줄 수 판단, 검토 항목을 수동으로 확인한다.
환경이 자동 조사를 허용하지 않거나, 한 단계씩 검증하고 싶을 때 쓴다.

> 두 모드는 같은 산출물(case YAML, index, keywords)을 만든다. 표현 방식만 다르다.

---

## 2. 시나리오 A — 첫 실행 (가장 흔한 케이스)

상황: RACH 실패가 의심되는 `.sdm` 로그 1건이 손에 있다. KG에는 아직 실제 case 0건.

| 단계 | 누가 | 행동 | 산출물 |
|---|---|---|---|
| 1 | 사람 | P0 프롬프트 입력 | 환경 진단 보고서 (입력 로그·L1SW 경로·가능여부) |
| 2 | 사람 | L1SW 실행 (P1이 명령 제시) | `_l1sw.txt` |
| 3 | Claude Code | P1 프롬프트 | `_l1sw.txt` 형식 명세 (cptime 포맷 확정) |
| 4 | Claude Code | P3 프롬프트, issue_type=rach_failure | `signals/<date>_rach_failure_..._signal.txt` + signature 등장 횟수 |
| 5 | Claude Code | P4 프롬프트 | `cases/<slug>_rach_failure_001.yaml` (신규, fingerprint 포함) + index.md 갱신 |
| 6 | 사람 | P5 프롬프트 → "승인" | review.status=reviewed, sequence_status=confirmed |
| 7 | Claude Code | P6 프롬프트 | 확인된 candidate signature → confirmed 승격, keywords.yaml v0.2 |

**사람 실질 입력:** P0 한 번, L1SW 실행, P5 "승인", P6 표 확인. 끝.

---

## 3. 시나리오 B — 같은 장애 재발 (fingerprint 매칭)

상황: 다른 단말 시험에서 같은 원인의 RACH 실패가 또 발생. wall-clock 시각은 당연히 다름.

```text
P3 (signal 생성) → P4 (분석)
   └ P4가 fingerprint(signature_set + sequence) 로 기존 cases/ 와 비교
      └ 일치 판정 → 신규 YAML 미생성
         └ 기존 case: occurrence_count++ , recent_occurrences 추가(최근 5건),
                       last_seen 갱신
```

핵심: **시각이 달라도 같은 장애로 인식된다.** time_range 가 아니라 cptime 기반 sequence +
signature_set 으로 매칭하기 때문. case 본체는 재발마다 증식하지 않는다.

**사람 실질 입력:** P4 결과의 "재발 매칭됨" 판정만 확인.

---

## 4. 시나리오 C — 담당영역 밖 (unresolved 이관)

상황: 분석해보니 RF 캘리브레이션 쪽 문제로 보임. 내 담당 범위가 아님.

```text
P4 (분석)
   └ "담당영역 밖" 판정
      └ cases/unresolved/<date>_<type>_<seq>_PENDING.yaml 생성
         - status: unresolved
         - handoff 블록 필수 (reason / suspected_domain / suspected_owner)
         - root_cause / fix / review = null  ← "분석자 미입력" 신호
```

이관받은 담당자가 root_cause/fix/review 를 채운 뒤:

```text
fingerprint 재매칭
   ├─ 기존 case 일치 → 병합(occurrence_count++), PENDING 폐기
   └─ 불일치        → 정식 case_id 부여, cases/ 로 승격
```

주의: **unresolved ≠ 원인 모름.** 원인이 불확실하지만 담당영역 안이면
`status: analyzed` + `confidence: low` 로 둔다 (unresolved 아님).

---

## 5. 핵심 규칙 요약 (schema v2)

분석 시 Claude Code 가 반드시 지키는 것:

```text
✔ case_id = <fingerprint-slug>_<issue_type>_<3-digit-seq>   (날짜 기반 금지)
✔ fingerprint 블록 필수: signature_set + sequence + sequence_status
✔ sequence 는 cptime 기준 상대순서 (절대시각 미포함)
✔ 위치 표기는 cptime_range 만 (line_range / time_range / raw_examples 금지)
✔ Jira 는 recent_occurrences[].jira 에만 (related.jira 없음)
✔ generic signature(RACH/SCG/RLC/HARQ)는 signature_set 에서 제외
✔ crash/dump case 생성 금지 (L1SW 전담)
✔ signature ID 는 keywords.yaml 네임스페이스만 사용
```

위반 시 대응표는 `scripts/README.md` 15장 참조.

---

## 6. 파일이 어디서 무엇을 하는가

```text
RCA_standalone/
├─ HANDOFF.md                      ← 새 세션 시작 시 가장 먼저 읽음 (설계·이력)
├─ USAGE_SCENARIO.md               ← 이 문서. 전체 사용 흐름
├─ prompt/
│  ├─ CLAUDE_CODE_PROMPTS_e2e_v1.md  ← ★ 단계별 실행 프롬프트 P0~P6 (모드 A)
│  ├─ 5_5_..._prompt.md              ← 5.5 구현 프롬프트 기본틀 (Step1/Step2)
│  └─ deepdive/                       ← 설계 논의·결정 이력
├─ scripts/
│  ├─ README.md                       ← Step1~9 수동 운영 가이드 (모드 B)
│  └─ *_prefilter.ps1                 ← 임시 signal 추출 (keywords.yaml과 R4에서 동기화)
└─ rca_kg/
   ├─ keywords.yaml                   ← ★ Signature SSOT (signature ID 단일 출처)
   ├─ signals/                        ← _l1sw.txt / signal 파일 (분석 입력)
   ├─ cases/                          ← case YAML (+ unresolved/ PENDING)
   ├─ skills_seed/                    ← issue_type별 분석 checklist
   ├─ indexes/index.md               ← fingerprint 기준 검색 index
   └─ schema/                         ← rca_case.schema.yaml v2, taxonomy, keywords.schema
```

---

## 7. 지금 상태와 막힌 지점 (2026-06-22)

| 레이어 | 상태 |
|---|---|
| 설계/스키마 (schema v2, taxonomy) | ✅ 준비됨 |
| Signature SSOT (keywords.yaml v0.1) | ✅ 준비됨 (대부분 candidate, confirmed 1개) |
| 분석 프롬프트 (P0~P6) | ✅ 준비됨 (이번 추가) |
| 입력 데이터 (signals/) | ⚠ 비어 있음 — 실로그 E2E 미수행 (최대 갭) |
| L1SW manifest fragment | ⛔ 미작성 (R4, P2로 자동 생성 예정) |
| 키워드 검증 | ⚠ confirmed 1건 외 전부 실로그 검증 전 |

**다음 한 걸음:** 사내에서 P0 → P1 → P2 를 돌려 L1SW 형식 확정 + manifest fragment 후보 생성,
이어서 실로그 1건으로 P3 → P4 → P5 E2E 1회 완주. 이게 끝나면 "최대 갭"이 닫힌다.

---

## 8. 환경 제약

```text
- L1SW 실행 / PowerShell pre-filter : 사내 Claude Code(PC) 전용 (모바일·외부 불가)
- Confluence write/update          : 미확인 → 구현 보류, export 파일로 대체
- Jira                             : read/create 가능 (MCP 연결됨)
- Perforce 연동                     : 향후 확장 후보
```
