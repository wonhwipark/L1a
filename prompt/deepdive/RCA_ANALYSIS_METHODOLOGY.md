# RCA 원인분석 방법론 — L1SW 출력에서 root_cause 도출하기

작성일: 2026-06-23 KST
적용 대상: 5.5 RCA Knowledge Graph 분석 단계 (E2E 프롬프트 P4, scripts/README §8)
선행 문서: `L1SW_AND_FLOW_20260620.md`(L1SW가 멈추는 6개 공백), `RCA_FEEDBACK_LOOP_DESIGN_20260622_KST.md`(워크플로)

---

## 0. 이 문서가 푸는 문제

기존 자료는 **워크플로**(root_cause를 어디에 쓰는지, confidence를 어떻게 표기하는지,
unresolved를 언제 분리하는지)는 정의했지만, **"L1SW가 뱉은 `_l1sw.txt`에서 실제로 어떻게
원인을 찾아내는가"** 하는 추론 방법은 비어 있었다. 이 문서가 그 공백을 채운다.

skills_seed의 issue_type별 체크리스트는 "무엇을 볼지"(evidence 목록)는 주지만 "어떤 순서로
어떻게 판단할지"(추론 절차)는 주지 않는다. 이 문서는 **issue_type에 무관한 공통 추론
엔진**이고, skills_seed는 그 엔진에 꽂는 issue_type별 단서 사전이다.

핵심 목표 두 가지:
- **성공률**: 틀린 root_cause를 confirmed로 올리는 일을 줄인다 (false positive 억제).
- **효율**: 700MB 로그를 무작정 읽지 않고, 가설을 먼저 세워 **확인할 줄만 골라 본다**.

---

## 1. 대원칙 — 가설 주도(hypothesis-driven), 증거 역추적

원인분석은 "로그를 처음부터 읽는" 작업이 아니다. 그렇게 하면 700MB는 물론 5MB
`_l1sw.txt`도 다 못 읽는다. 대신:

```
증상(symptom) → 가설 후보 N개 → 각 가설이 참이라면 로그에 있어야 할 증거 →
그 증거를 cptime 구간에서 확인/반증 → 남은 가설이 root_cause
```

이것은 의료 감별진단과 같다. "열이 난다"에서 모든 검사를 하지 않고, 가능한 병 몇 개를
세운 뒤 그것을 가르는 검사만 한다. **로그 grep은 검사이고, signature는 증상이다.**

> 왜 효율적인가: L1SW가 이미 모듈·시간으로 1차 필터한 `_l1sw.txt`에서, 5.5는 **가설을
> 가르는 signature만** 추가로 grep한다. 전체 정독이 아니라 표적 확인이다.

---

## 2. 7단계 분석 절차 (issue_type 공통)

### S1. 증상 고정 (Symptom anchoring)
- `_l1sw.txt`에서 **실패를 단정하는 signature**(role: `failure_event`)의 **첫 등장
  cptime**을 찾는다. 이게 분석의 원점(anchor)이다.
- 예: RACH면 `RACH_PHY_TIMER_EXPIRY`/`RACH_RAR_TIMEOUT`, SCG면 `SCG_RLF`/`SCG_T310`.
- failure_event가 여러 개면 **가장 이른 것**을 원점으로. 뒤의 것은 연쇄 결과일 수 있다.

### S2. 시간창 설정 (Window around anchor)
- anchor cptime 기준 **앞으로** 충분히(예: 진입 이벤트까지), **뒤로** 약간(fallback/해제
  동작까지) 구간을 잡는다. 이 구간이 `cptime_range` 후보가 된다.
- 앞을 보는 이유: **원인은 항상 결과보다 먼저 cptime에 찍힌다.** 뒤를 보는 이유: 진짜
  실패인지(복구 못 함) 일시적 경고인지(곧 성공) 구분.

### S3. 정상 경로 대조 (Happy-path delta)
- 해당 issue_type의 **정상 시퀀스**를 기준선으로 둔다.
  - RACH: `Msg1→Msg2(RAR)→Msg3→Msg4(contention resolution)`
  - SCG:  `measReport→SCG-Config→randomAccess→SCG 활성`
- anchor 구간의 실제 시퀀스를 정상 시퀀스와 겹쳐, **어느 단계에서 처음 이탈했는가**를
  특정한다. **최초 이탈 지점(first divergence)이 root_cause에 가장 가깝다.**
- 예: `Msg1→Msg2→Msg3→(타임아웃)`이면 Msg3 이후 응답 단계 이탈 → Msg4/contention 또는
  상위 grant 문제로 가설이 좁혀진다.

### S4. 가설 후보 나열 (Differential)
- skills_seed의 **Root Cause Categories 표**를 가설 풀(pool)로 사용한다.
- S3의 최초 이탈 지점과 **양립 가능한 category만** 남긴다. (이탈이 Msg3 전이면
  `scheduler_grant_missing`은 후보에서 빠진다 — grant는 Msg3 이후 단계.)
- 보통 2~4개로 좁혀진다. 1개로 즉단하지 말 것(확증 편향).

### S5. 가설 가르기 (Discriminating evidence)
- 각 가설을 **참/거짓으로 가르는 단 하나의 결정적 단서**를 정하고 그것만 확인한다.
  | 가설 | 가르는 단서(있으면 ↑, 없으면 ↓) |
  |---|---|
  | radio_access_timeout | 동일 cell 타 UE는 RACH 성공? → 성공이면 단일 UE 무선문제 ↑ |
  | preamble_collision | 재시도 시 동일 preamble 반복? |
  | scheduler_grant_missing | Msg3 직전 UL grant 로그 존재? |
  | config_parameter_mismatch | timeout 값이 규격 대비 비정상으로 짧음? |
  | api_contract_mismatch | REQ/CNF 파라미터 불일치 로그? |
- **하나의 grep으로 두 가설을 동시에 가를 수 있으면 그것부터** 한다(효율).

### S6. 단일 원인 vs 환경 요인 분리 (Scope test)
- "이 UE만의 문제"인지 "cell/환경 문제"인지 가른다.
  - 동일 cell 내 **다른 UE가 같은 시간대에 성공**했나? → 성공이면 단말 측, 전부 실패면
    cell/네트워크/RF 측.
- **환경/RF 측으로 판정되면 → 담당영역 밖 → `unresolved` + handoff** (원인 미상이 아니라
  "내 도메인 아님"). 이 분기를 S6에서 명시적으로 건다.

### S7. 인과사슬 확정 + confidence 산정
- 남은 가설을 **인과사슬**로 적는다(증상 아님, 원인→결과 연결):
  `UL grant 누락 → Msg3 미전송 → RAR 재시도 → PHY_TIMER_EXPIRY → RACH 실패`
- 사슬의 **모든 화살표가 로그 증거로 뒷받침되면** confidence ↑.
  끊긴 화살표(추정으로 메운 곳)가 있으면 그 수만큼 confidence ↓.

---

## 3. confidence 산정 기준 (자의적 표기 금지)

`root_cause.confidence`는 느낌이 아니라 **증거 충족도**로 정한다.

| confidence | 조건 |
|---|---|
| `confirmed` | 인과사슬 전 구간 로그 증거 존재 + 가설 가르는 단서 명확 + 재현/타UE 대조 완료 |
| `high` | 인과사슬 대부분 증거 존재, 화살표 1개만 합리적 추정 |
| `medium` | 최초 이탈 지점은 특정됐으나 그 위 원인은 2개 가설이 남음 |
| `low` | 증상은 분류됐으나 최초 이탈 지점 위로 단서 부족 (담당영역 안, 추가 분석 필요) |

> **중요한 구분**(FEEDBACK_LOOP §11 재확인):
> - 담당영역 안 + 원인 불확실 = `status: analyzed` + `confidence: low`
> - 담당영역 밖 = `status: unresolved` + handoff (confidence 표기 안 함)
> 둘을 섞지 말 것.

---

## 4. 효율 규칙 (큰 로그에서 시간 낭비 안 하기)

1. **anchor부터, 바깥으로**: 파일 처음부터 읽지 말고 failure_event cptime을 먼저 찾아
   거기서 양방향으로 확장한다.
2. **가설이 부르는 grep만**: S5의 "가르는 단서"에 해당하는 signature만 추가 검색한다.
   전체 signature를 다 세지 않는다.
3. **generic은 위치 좁힐 때만**: `RACH`/`SCG`/`RLC`/`HARQ`(noise_risk:high)는 원인 판정에
   쓰지 말고 "이 구간이 맞나" 문맥 확인용으로만. fingerprint·root_cause 근거로 금지.
4. **cptime 단조성 활용**: 원인은 결과보다 cptime이 작다. 큰 cptime 줄을 원인 후보로
   올리지 말 것.
5. **3회 규칙**: 같은 가설을 가르려고 3번 넘게 grep했는데 결론이 안 나면, 그 가설은
   `low`로 두고 다음으로 넘어간다(무한 정독 방지).

---

## 5. 실패를 부르는 안티패턴 (이걸 하면 성공률이 떨어진다)

- **증상을 원인으로 적기**: "PHY_TIMER_EXPIRY 발생"은 증상이다. 원인은 "왜 타이머가
  만료됐는가"다. root_cause.summary에 signature 이름만 적으면 실패.
- **첫 가설 확증**: 처음 떠오른 category의 증거만 찾고 반증을 안 본다. S4에서 최소 2개를
  강제로 세우는 이유.
- **연쇄결과를 원점으로**: 늦게 찍힌 failure_event(예: 연결 해제)를 anchor로 잡으면 진짜
  최초 이탈을 놓친다. S1에서 "가장 이른 failure_event".
- **wall-clock으로 순서 판단**: 시험마다 시계가 다르다. 순서는 **항상 cptime**.
- **담당영역 밖인데 low로 우기기**: RF/환경 문제를 `confidence:low`로 들고 있으면 영원히
  안 풀린다. S6에서 unresolved로 빠르게 넘긴다.
- **generic signature로 fingerprint**: false 매칭 양산. §4-3 위반.

---

## 6. 분석 결과를 case에 매핑 (schema v2 연결)

| 분석 산출 | case 필드 |
|---|---|
| S1 anchor signature | `fingerprint.signature_set`(ID), `log_patterns[].signature` |
| S2 시간창 | `log_patterns[].cptime_range` |
| S3 최초 이탈 시퀀스 | `fingerprint.sequence` (cptime 정렬, ID 표기, 절대시각 금지) |
| S4~S5 채택 category | `root_cause.category` (taxonomy active 목록) |
| S7 인과사슬 | `root_cause.summary` (사슬 문장) |
| S7 confidence | `root_cause.confidence` (§3 기준) |
| S5 가르는 단서 줄 | `root_cause.evidence` → `log_patterns[].pattern_id` |
| S6 환경 판정 | `unresolved/` 분리 + `handoff` 블록 |

> signature는 **반드시 keywords.yaml ID**(`RACH_MSG1` 등, alias 금지). 2026-06-23 정합성
> 통일과 일치.

---

## 7. 한 줄 요약

```
증상 anchor 고정 → 시간창 → 정상경로 대조로 최초 이탈 특정 → 가설 2~4개 →
가르는 단서만 grep → 단말/환경 분리(환경이면 handoff) → 인과사슬+confidence
원칙: 가설이 부르는 줄만 본다. 증상이 아니라 원인을 적는다. 순서는 cptime.
```

이 절차는 issue_type과 무관하다. skills_seed의 issue_type별 단서 표를 §2의 S3~S5에
꽂아 쓰면 4종(rach/scg/tx/l2) 모두 동일 엔진으로 분석된다.
