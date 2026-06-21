# L1 로그 분석 실습 가이드

> 대상: RACH failure / SCG failure 등 L1 로그 분석  
> 환경: Windows 11, PowerShell, VSCode + Claude Code 확장  
> 예상 소요: 처음 설정 10분 + 분석 1건당 15~30분

---

## 이 가이드가 하는 일

대용량 로그(100MB~1GB)를 직접 AI에 붙여넣는 대신, 아래 3단계로 분석합니다.

```
① 로그 파일 (100MB~1GB)
      ↓  [PowerShell 스크립트 실행 — 사람이 함]
② signal 파일 (수 MB로 축소, 관련 구간만 추출)
      ↓  [Claude Code 에 분석 요청 — 사람이 요청, Claude 가 실행]
③ rca_kg/cases/YYYY-MM-DD_xxx_001.yaml  (분석 결과 자동 저장)
      ↓  [사람이 검토]
④ review.status: draft → reviewed → confirmed
```

---

## 준비물 체크리스트

시작 전 아래를 모두 확인하세요.

- [ ] 분석할 로그 파일이 있다 (예: `service.log`)
- [ ] VSCode 가 설치되어 있다
- [ ] VSCode 에 **Claude Code 확장**이 설치되어 있다
- [ ] PowerShell 터미널을 열 수 있다 (Windows 기본 제공)
- [ ] 이 저장소 폴더가 열려 있다: `d:\User\whpark\L1_AI_Automation\L1a\`

---

## 처음 한 번만 하는 설정

### PowerShell 스크립트 실행 허용

Windows 기본 설정은 PowerShell 스크립트 실행을 막습니다. 처음 한 번만 아래를 실행합니다.

1. `Win + R` → `powershell` 입력 → Enter (일반 PowerShell)
2. 아래 명령어를 붙여넣고 Enter:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

3. `Y` 입력 후 Enter
4. 터미널 닫기

---

## Step 1 — Pre-filter 실행 (PowerShell)

### 1-1. PowerShell 터미널 열기

**방법 A — VSCode 터미널 (권장):**
1. VSCode 상단 메뉴 → `Terminal` → `New Terminal`
2. 터미널 창이 화면 아래에 열림

**방법 B — 독립 PowerShell:**
1. `Win + R` → `powershell` → Enter

### 1-2. scripts 폴더로 이동

아래 명령어를 터미널에 붙여넣고 Enter:

```powershell
cd "d:\User\whpark\L1_AI_Automation\L1a\scripts"
```

현재 위치 확인 (선택):
```powershell
pwd
# 결과: d:\User\whpark\L1_AI_Automation\L1a\scripts  ← 이렇게 나와야 함
```

### 1-3. 스크립트 실행

**로그 파일 경로 확인 먼저:**
- 탐색기에서 로그 파일 위치를 확인합니다
- 경로 예시: `C:\work\logs\20260620\service.log`

---

**RACH failure 분석 시:**

```powershell
.\rach_failure_prefilter.ps1 -InputLog "C:\work\logs\20260620\service.log" -OutputTxt "..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt"
```

> `2026-06-20_rach_failure_001` 부분을 **오늘 날짜와 순번**으로 바꾸세요.  
> 예: 두 번째 분석이면 `_002`, 세 번째면 `_003`

---

**SCG failure 분석 시:**

```powershell
.\scg_failure_prefilter.ps1 -InputLog "C:\work\logs\20260620\service.log" -OutputTxt "..\rca_kg\signals\2026-06-20_scg_failure_001_signal.txt"
```

---

**경로 입력이 번거로울 때 — 탐색기에서 드래그:**
1. 탐색기에서 로그 파일 선택
2. Shift + 우클릭 → "경로로 복사"
3. 명령어의 `"C:\work\..."` 부분에 붙여넣기

### 1-4. 실행 결과 확인

정상 실행 시 아래처럼 출력됩니다:

```
입력: C:\work\logs\20260620\service.log
키워드: PHY_TIMER_EXPIRY, RACH, preamble, RAR, Msg1, Msg2, Msg3, Msg4, rach_fail, RA-RNTI
전후 context: 20 줄
완료: ..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt (3842 줄)
```

**줄 수 기준:**

| 줄 수 | 판단 | 조치 |
|-------|------|------|
| 0줄 | 키워드가 로그에 없음 | 아래 "문제 발생 시" 참조 |
| 100줄 미만 | 너무 적음 | `-Context 50` 으로 늘리기 |
| 1,000~50,000줄 | 적당 | 그대로 진행 |
| 100,000줄 이상 | 너무 많음 | `-Context 5` 로 줄이기 |

**줄 수 조정 방법:**
```powershell
# context 를 50줄로 늘리기 (기본값 20)
.\rach_failure_prefilter.ps1 -InputLog "C:\..." -OutputTxt "..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt" -Context 50

# context 를 5줄로 줄이기
.\rach_failure_prefilter.ps1 -InputLog "C:\..." -OutputTxt "..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt" -Context 5
```

### 1-5. signal 파일 생성 확인

```powershell
# signal 파일이 생성됐는지 확인
ls ..\rca_kg\signals\

# 첫 30줄 미리보기
Get-Content ..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt -TotalCount 30
```

---

## Step 2 — Claude Code 에 분석 요청

### 2-1. Claude Code 채팅창 열기

**VSCode Claude Code 확장 사용 시:**
1. VSCode 좌측 사이드바에서 Claude 아이콘 클릭
2. 채팅창이 열림

**Claude Code CLI 사용 시:**
1. VSCode 터미널에서 `claude` 입력 → Enter

### 2-2. 분석 요청 프롬프트 (복사해서 사용)

아래 텍스트를 복사 → Claude Code 채팅창에 붙여넣기 → 날짜/순번 수정 후 전송:

```
@rca_kg/signals/2026-06-20_rach_failure_001_signal.txt
@rca_kg/skills_seed/rach_failure_analyzer.md
@rca_kg/cases/EXAMPLE_rach_failure_001.yaml
@rca_kg/schema/rca_case.schema.yaml

위 signal 파일을 분석해줘.
분석 기준은 rach_failure_analyzer.md 의 Analysis Checklist 를 모두 확인해줘.
결과는 EXAMPLE_rach_failure_001.yaml 형식에 맞춰
rca_kg/cases/2026-06-20_rach_failure_001.yaml 로 저장해줘.
저장 후 rca_kg/indexes/index.md Case 목록 표에 한 줄 추가해줘.
```

> `2026-06-20_rach_failure_001` 부분을 Step 1 에서 쓴 날짜/순번과 동일하게 맞추세요.

**SCG failure 분석 시 (복사해서 사용):**

```
@rca_kg/signals/2026-06-20_scg_failure_001_signal.txt
@rca_kg/skills_seed/scg_failure_analyzer.md
@rca_kg/cases/EXAMPLE_rach_failure_001.yaml
@rca_kg/schema/rca_case.schema.yaml

위 signal 파일을 분석해줘.
분석 기준은 scg_failure_analyzer.md 의 Analysis Checklist 를 모두 확인해줘.
결과는 EXAMPLE yaml 형식에 맞춰
rca_kg/cases/2026-06-20_scg_failure_001.yaml 로 저장해줘.
저장 후 rca_kg/indexes/index.md Case 목록 표에 한 줄 추가해줘.
```

### 2-3. Claude Code 가 자동으로 하는 것

분석 요청을 받으면 Claude Code 가 자동으로:

1. signal 파일 읽기
2. skills_seed checklist 기준으로 분석
3. `rca_kg/cases/2026-06-20_rach_failure_001.yaml` 파일 생성
4. `rca_kg/indexes/index.md` 에 한 줄 추가

> 사람이 직접 파일을 만들거나 복사할 필요가 없습니다.

### 2-4. 분석 중 추가로 물어볼 수 있는 것

분석 결과를 보고 더 자세히 알고 싶으면 후속 질문 가능:

```
RAR timeout 발생 횟수를 세어줘
```
```
동일 UE 기준 RACH 재시도 전체 타임라인을 표로 정리해줘
```
```
confidence 를 medium 으로 설정한 근거를 설명해줘
```
```
Root Cause Category 를 radio_access_timeout 으로 특정한 이유는?
```

---

## Step 3 — 결과 검토 및 승인

### 3-1. 생성된 YAML 파일 열기

VSCode 탐색기에서:
```
rca_kg/
  └─ cases/
       └─ 2026-06-20_rach_failure_001.yaml  ← 이 파일 클릭
```

### 3-2. 검토 항목 체크리스트

| 항목 | 확인 내용 |
|------|----------|
| `symptom.occurred_at` | 발생 시각이 맞는지 |
| `root_cause.category` | `taxonomy.yaml` 의 카테고리 중 하나인지 |
| `root_cause.confidence` | low/medium/high/confirmed 기준 적절한지 |
| `log_patterns.raw_examples` | 실제 로그 원문과 일치하는지 |
| `log_patterns.line_range` | 원본 로그 라인 번호가 맞는지 |
| `prevent_rule` | 실제로 적용 가능한 방어 규칙인지 |

### 3-3. review.status 업데이트

검토 완료 후 YAML 파일에서:

```yaml
review:
  status: draft      # ← 이 부분을 reviewed 로 변경
  reviewer: ""       # ← 이름 입력
  reviewed_at: ""    # ← 날짜 입력 (예: "2026-06-20T10:00:00+09:00")
```

변경 후:

```yaml
review:
  status: reviewed
  reviewer: "whpark"
  reviewed_at: "2026-06-20T10:00:00+09:00"
```

### 3-4. Jira / HLD / TC 연결 (있는 경우)

```yaml
related:
  jira:
    - "L1-1234"      # ← 관련 Jira 티켓
  hld: []
  tc: []
```

---

## Step 4 — skills_seed 업데이트 (선택, 새 패턴 발견 시)

분석 중 기존 checklist 에 없던 새 패턴을 발견했으면 Claude Code 에 요청:

```
rach_failure_analyzer.md 의 누적 패턴 표에 오늘 케이스를 한 줄 추가해줘.
case_id: 2026-06-20_rach_failure_001
root_cause: radio_access_timeout
confidence: medium
```

> 10건 이상 쌓이면 공통 패턴이 보이기 시작합니다. 그 때 checklist 를 보강하세요.

---

## 자주 쓰는 PowerShell 명령 모음

```powershell
# signal 파일 줄 수 확인
(Get-Content ..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt | Measure-Object -Line).Lines

# signal 파일에서 특정 키워드가 있는 줄만 다시 보기
Select-String -Path "..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt" -Pattern "PHY_TIMER_EXPIRY"

# signal 파일 처음 50줄 확인
Get-Content "..\rca_kg\signals\2026-06-20_rach_failure_001_signal.txt" -TotalCount 50

# cases 폴더 목록 확인
ls ..\rca_kg\cases\

# 오늘 생성된 case 파일 확인
ls ..\rca_kg\cases\ | Where-Object { $_.LastWriteTime -gt (Get-Date).Date }
```

---

## 문제 발생 시

| 증상 | 가능한 원인 | 해결 방법 |
|------|------------|-----------|
| `입력 파일 없음` 오류 | 로그 파일 경로 오타 | 경로를 `"..."` 따옴표로 감싸기, 탐색기에서 경로 복사 사용 |
| signal 파일이 0줄 | 키워드가 로그에 없음 | ps1 파일의 `$keywords` 에 실제 로그 키워드 추가 후 재실행 |
| signal 파일이 너무 큼 | Context 가 너무 넓음 | `-Context 5` 로 줄이기 |
| signal 파일이 너무 적음 | Context 가 너무 좁음 | `-Context 50` 으로 늘리기 |
| 스크립트 실행 안 됨 | PowerShell 실행 정책 | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` 실행 |
| `@파일경로` 가 Claude 에 안 읽힘 | 경로 오타 또는 파일 미생성 | `ls ..\rca_kg\signals\` 로 파일 존재 확인 |
| Claude 가 YAML 저장 안 함 | 권한 또는 경로 문제 | Claude Code 에 "파일 쓰기를 허용해줘" 권한 승인 클릭 |

---

## 파일 역할 한눈에 보기

```
L1a/
  ├─ scripts/
  │    ├─ README.md                        ← 이 파일 (실습 가이드)
  │    ├─ rach_failure_prefilter.ps1       ← RACH 로그 추출 스크립트
  │    └─ scg_failure_prefilter.ps1        ← SCG 로그 추출 스크립트
  │
  └─ rca_kg/
       ├─ signals/                         ← ① pre-filter 출력 (사람이 폴더 지정)
       │    └─ YYYY-MM-DD_xxx_001_signal.txt
       │
       ├─ cases/                           ← ③ RCA 결과 YAML (Claude 가 자동 생성)
       │    ├─ EXAMPLE_rach_failure_001.yaml   (작성 형식 참고용 템플릿)
       │    └─ YYYY-MM-DD_xxx_001.yaml
       │
       ├─ skills_seed/                     ← 분석 기준 가이드 (반복 시 업데이트)
       │    ├─ rach_failure_analyzer.md    ← RACH 분석 checklist
       │    ├─ scg_failure_analyzer.md
       │    ├─ tx_abnormal_analyzer.md
       │    ├─ l2_max_retransmission_analyzer.md
       │    └─ crash_analyzer.md
       │
       ├─ indexes/
       │    └─ index.md                    ← ③ 전체 케이스 목록 (Claude 가 자동 추가)
       │
       └─ schema/
            ├─ rca_case.schema.yaml        ← YAML 필드 정의 (참고용)
            └─ taxonomy.yaml               ← issue_type / confidence 기준 (참고용)
```

**사람이 직접 하는 것:**
- pre-filter 스크립트 실행 (Step 1)
- Claude Code 에 분석 요청 (Step 2)
- 결과 검토 및 `review.status` 업데이트 (Step 3)
- Jira/HLD/TC 연결 정보 입력
- 키워드가 틀렸을 때 ps1 파일 수정

**Claude Code 가 자동으로 하는 것:**
- signal 파일 읽고 분석
- `rca_kg/cases/<case_id>.yaml` 생성
- `rca_kg/indexes/index.md` 한 줄 추가
- skills_seed 업데이트 후보 제안

---

## 참고 — keywords 수정 방법

실제 로그를 보고 키워드가 맞지 않으면 ps1 파일의 `$keywords` 블록을 수정합니다.

```powershell
# VSCode 에서 파일 열기
code .\rach_failure_prefilter.ps1
```

파일 안에서 수정할 위치:

```powershell
$keywords = @(
    "PHY_TIMER_EXPIRY",     # 확인됨  ← 실제 로그에서 확인된 키워드
    "RACH",                 # 추정    ← 아직 확인 안 된 키워드
    "preamble",             # 추정
    ...
)
```

- `# 확인됨`: 실제 로그에서 등장을 확인한 키워드
- `# 추정`: 아직 확인하지 못한 키워드 — 확인 후 `# 확인됨` 또는 삭제
