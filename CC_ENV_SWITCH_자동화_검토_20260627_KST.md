# CC_ENV_SWITCH 분리 및 자동화 검토

작성일: 2026-06-27 KST
원본: `RCA_standalone/prompt/CLAUDE_CODE_ENV_SWITCH_PROMPTS.md` (245줄)

---

## 1. 현재 문서 진단

### 1.1 성격

| 항목 | 현재 | CodeAnalyzer 5.2 비교 |
|------|------|----------------------|
| 유형 | 프롬프트 레시피 (사람이 읽고 붙여넣기) | 스킬 패키지 (Claude Code가 읽고 실행) |
| 실행 주체 | 사람 → Claude Code 에 프롬프트 전달 | Claude Code가 SKILL.md 규칙대로 자율 수행 |
| 산출물 | cc-switch.ps1 + 프로파일 JSON (매번 재생성) | 분석 결과 .md + structure.json (반복 가능) |
| 상태 추적 | 없음 (사람이 STEP 1→2 순서 기억) | NEXT_STEP + current_slug + Phase 0→N→F |

### 1.2 워크플로우 분석

```
현재 흐름 (수동 6단계):

 [리눅스 PC]                    [사람]                     [윈도우 PC]
     │                            │                            │
     │ ◀── STEP1 프롬프트 붙여넣기 ──│                            │
     │──── 이식카드 출력 ──────────▶│                            │
     │                            │── 토큰 확보 (사내 절차) ──▶│   │
     │                            │── STEP2 프롬프트 + 카드 ──▶│   │
     │                            │                            │── cc-switch.ps1 생성
     │                            │                            │── 프로파일 JSON 생성
     │                            │                            │── 수동 검증
```

**병목 3곳:**
- ❶ Claude Code가 cc-switch.ps1을 **매번 처음부터 생성** (245줄 프롬프트 의 ~60%가 스크립트 요구사항)
- ❷ 이식 카드가 **자유 텍스트** → STEP 2에서 파싱 불안정
- ❸ 검증이 사람 수동 (`/status` 확인, 새 터미널 열기)

---

## 2. 자동화 가능성 판정

### 2.1 자동화 가능 (스크립트/템플릿으로 대체)

| # | 현재 (프롬프트 지시) | 자동화 방안 | 효과 |
|---|---------------------|-------------|------|
| A | STEP 2에서 cc-switch.ps1 생성 지시 (S1~S6 요구사항 60줄) | **사전 제작된 cc-switch.ps1 배포** — 파라메트릭 설계, 환경 변수만 주입 | 프롬프트 60% 제거 |
| B | 프로파일 JSON 골격을 프롬프트 안에 예시로 기술 | **aws.settings.template.json / corp.settings.template.json 파일 배포** — placeholder(`${PLACEHOLDER}`)만 사용자 교체 | 프롬프트 15% 제거 |
| C | $PROFILE 등록, setx 명령 등 수동 안내 | **install_cc_switch.ps1** 1회 실행 스크립트 — 디렉토리 생성, 골든 스냅샷, $PROFILE 등록 자동 | 설치 원터치 |
| D | 전환 후 수동 `/status` 확인 | cc-switch.ps1 내 **S5 자동 검증** (이미 설계에 있음 — 구현만 하면 됨) | 검증 자동 |
| E | 비상 복구 카드를 프롬프트 출력에 의존 | **EMERGENCY_CARD.md 파일 사전 배포** | 항상 존재, 인쇄 가능 |

### 2.2 반자동 (사람 입력 1회 필요, 이후 자동)

| # | 항목 | 이유 | 자동화 범위 |
|---|------|------|------------|
| F | 이식 카드 작성 | 리눅스 머신에서 실행해야 함 (크로스 머신) | STEP 1 프롬프트는 유지하되, 출력 형식을 **YAML 강제** → 윈도우에서 파일로 읽기 |
| G | 토큰/시크릿 입력 | 보안상 사람이 직접 입력 | `setx CORP_ANTHROPIC_TOKEN "___"` 한 줄만 남김 |

### 2.3 자동화 불가

| # | 항목 | 이유 |
|---|------|------|
| H | 사내 토큰 발급 | 사내 인증 절차 (외부 자동화 불가) |
| I | 사내 게이트웨이 호환성 최초 판단 | 환경 의존, 1회성 |
| J | 리눅스 PC 접근 | 물리적/VPN 접근 필요 |

---

## 3. 제안 패키지 구조

```
CC_ENV_SWITCH/
├── START_HERE.md                  ← 문서맵 + 3분 퀵스타트
├── NEXT_STEP.md                   ← 진행 상태 추적 (slug auto-carry 적용)
│
├── prompt/
│   ├── STEP1_EXTRACT.md           ← 리눅스용 프롬프트 (copy-paste 유지)
│   │                                 변경: 출력을 transplant_card.yaml 형식으로 강제
│   └── README.md                  ← STEP 1만 프롬프트로 남는 이유 설명
│
├── templates/
│   ├── aws.settings.template.json       ← Bedrock 프로파일 골격
│   ├── corp.settings.template.json      ← 사내 게이트웨이 프로파일 골격
│   └── transplant_card.template.yaml    ← 이식 카드 표준 형식 (STEP 1 출력 규격)
│
├── scripts/
│   ├── cc-switch.ps1              ← 사전 제작 완성 스크립트 (S1~S6 전체 구현)
│   ├── install_cc_switch.ps1      ← 1회 설치: 디렉토리/골든스냅샷/$PROFILE 등록
│   ├── apply_card.ps1             ← 이식 카드 YAML → corp 프로파일 자동 반영
│   └── verify_switch.ps1          ← 전환 상태 독립 검증
│
├── EMERGENCY_CARD.md              ← 비상 복구 5줄 (사전 배포, 항상 존재)
└── VERSION.md
```

### 3.1 CodeAnalyzer 패턴 적용 내역

| CodeAnalyzer 패턴 | ENV_SWITCH 적용 | 비고 |
|-------------------|-----------------|------|
| START_HERE 문서맵 | ✅ START_HERE.md 상단에 파일-역할 테이블 | — |
| slug auto-carry | ✅ NEXT_STEP.md의 `current_slug` 필드 | slug = `extract` → `install` → `configure` → `verify` |
| `▶ 다음 붙여넣기` 힌트 | ⚠️ 부분 적용 | STEP 1 프롬프트 끝에만 (나머지는 스크립트 자동 진행) |
| Phase 0→N→F | ❌ 불필요 | 4단계 선형 플로우, 분기/반복 없음 |
| session bootstrap | ❌ 불필요 | 스크립트 실행이므로 세션 불변 규칙 로딩 불요 |
| `_5.x` 접미사 (충돌방지) | ✅ 적용 가능 | 단 단독 패키지이므로 당장 불필요, 5.x 시리즈 합류 시 적용 |

---

## 4. 자동화 전후 비교

### Before (현재): 사람 6단계, 프롬프트 245줄

```
사람 조작 ─────────────────────────────────────────────────────
 1. STEP1 프롬프트 복사 → 리눅스 Claude Code 붙여넣기
 2. 이식 카드 텍스트 복사
 3. 토큰 확보
 4. STEP2 프롬프트 + 카드 → 윈도우 Claude Code 붙여넣기
 5. Claude Code가 스크립트/프로파일 생성 (대기)
 6. 수동 검증 (새 터미널 → /status × 2)
──────────────────────────────────────────────────────────────
 총 프롬프트 입력: ~170줄 (STEP1 45줄 + STEP2 125줄)
 Claude Code 생성 대기: cc-switch.ps1 전체 + JSON 2개 + 복구카드
```

### After (제안): 사람 3단계, 프롬프트 45줄

```
사람 조작 ─────────────────────────────────────────────────────
 1. STEP1 프롬프트 복사 → 리눅스 Claude Code 붙여넣기
    → 출력: transplant_card.yaml (구조화)
 2. 토큰 확보 + setx 1줄 실행
 3. install_cc_switch.ps1 실행 (원터치)
    → 자동: 디렉토리 생성, 골든 스냅샷, 프로파일 복사,
           $PROFILE 등록, 이식카드 적용, 검증
──────────────────────────────────────────────────────────────
 총 프롬프트 입력: ~45줄 (STEP1만)
 Claude Code 생성 대기: 0 (모두 사전 제작)
```

### 정량 비교

| 지표 | Before | After | 감소율 |
|------|--------|-------|--------|
| 사람 조작 단계 | 6 | 3 | **50%** |
| 프롬프트 입력량 | ~170줄 | ~45줄 | **74%** |
| Claude Code 생성 대기 | cc-switch.ps1 전체 (~200줄) | 0 | **100%** |
| 수동 검증 | 2회 (aws/corp 각각) | 0 (자동) | **100%** |
| 비상 복구 카드 | 생성 의존 (없을 수 있음) | 항상 존재 | — |

---

## 5. 핵심 설계 변경점

### 5.1 이식 카드 → YAML 구조화

**현재** (자유 텍스트):
```
── 사내(CORP) 이식 카드 ──────────────────
provider           : gateway
ANTHROPIC_BASE_URL : gw.corp.example.com
...
```

**제안** (transplant_card.yaml):
```yaml
# transplant_card.yaml — STEP 1 출력, STEP 3 입력
version: "1.0"
extracted_at: "2026-06-27T12:00:00+09:00"
source_machine: "linux-corp"

provider:
  type: "gateway"          # gateway | litellm | direct
  base_url_host: "gw.corp.example.com"
  auth_method: "bearer"    # bearer (AUTH_TOKEN) | api_key (API_KEY)
  model_alias: ""          # 게이트웨이 전용 모델명, 없으면 비움

env_keys:
  ANTHROPIC_BASE_URL: "https://gw.corp.example.com"
  ANTHROPIC_MODEL: ""
  CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS: "1"  # 필요 시
  NO_PROXY: ""

custom_headers: []         # 헤더 이름만 (값 제외)
notes: ""                  # 특이사항
```

**효과**: `apply_card.ps1`이 이 YAML을 파싱하여 `corp.settings.template.json`의 placeholder를 자동 치환 → 사람 개입 0.

### 5.2 cc-switch.ps1 사전 제작

현재 프롬프트의 **S1~S6 요구사항(60줄)**이 그대로 구현된 완성 스크립트를 패키지에 포함.
프롬프트에서 "이렇게 만들어줘"로 지시하던 것이 "이미 만들어져 있음"으로 전환.

변경 가능 파라미터는 스크립트 상단 `$Config` 블록으로 분리:
```powershell
$Config = @{
    ProfileDir   = "$env:USERPROFILE\.claude\profiles"
    GoldenDir    = "$env:USERPROFILE\.claude\profiles\_golden"
    BackupDir    = "$env:USERPROFILE\.claude\backups"
    SettingsPath = "$env:USERPROFILE\.claude\settings.json"
    MaxBackups   = 20
    VerifyCmd    = 'claude --version'  # 또는 'claude -p "reply OK"'
}
```

### 5.3 install_cc_switch.ps1 워크플로우

```
install_cc_switch.ps1 실행 시:

 1. profiles/, profiles/_golden/, backups/ 디렉토리 생성
 2. 현재 settings.json → _golden/settings.ORIGINAL.json (읽기전용)
 3. aws.settings.template.json → profiles/aws.settings.json
 4. corp.settings.template.json → profiles/corp.settings.json
 5. transplant_card.yaml 존재 시 → apply_card.ps1 호출 (corp 프로파일 자동 완성)
 6. cc-switch.ps1 → 지정 위치 복사
 7. $PROFILE에 cc-switch 함수 등록 (이미 있으면 스킵)
 8. cc-switch status 실행하여 설치 확인
```

**사람 입력**: `.\install_cc_switch.ps1` 한 줄.
**전제**: transplant_card.yaml과 토큰 환경변수가 준비된 상태.

---

## 6. NEXT_STEP.md 설계 (slug auto-carry)

```yaml
# NEXT_STEP.md — CC_ENV_SWITCH 진행 상태
current_slug: extract    # extract → install → configure → verify → done

slugs:
  extract:
    label: "STEP 1 — 리눅스에서 이식 카드 추출"
    action: "prompt/STEP1_EXTRACT.md 의 프롬프트를 리눅스 Claude Code에 붙여넣기"
    next: install
    output: "templates/transplant_card.yaml (사용자가 채운 것)"

  install:
    label: "설치 — install_cc_switch.ps1 실행"
    action: "scripts/install_cc_switch.ps1 실행"
    prereq: "transplant_card.yaml + CORP_ANTHROPIC_TOKEN 환경변수"
    next: configure

  configure:
    label: "프로파일 확인/미세 조정"
    action: "profiles/ 의 JSON 확인, 필요 시 수정"
    next: verify

  verify:
    label: "전환 검증"
    action: "cc-switch aws → 새 터미널 → /status, cc-switch corp → 새 터미널 → /status"
    next: done

  done:
    label: "완료"
    action: "일상 사용: cc-switch aws | cc-switch corp"
```

---

## 7. 판정 요약

| 판정 | 내용 |
|------|------|
| **분리 타당성** | ✅ RCA 5.5와 무관한 인프라 도구 — 독립 패키지로 분리 적절 |
| **자동화 가능 범위** | STEP 2 전체 (프롬프트 125줄) → 스크립트 3개로 대체 가능 |
| **자동화 불가 범위** | STEP 1 (크로스 머신), 토큰 확보 (보안), 게이트웨이 호환 판단 (1회) |
| **CodeAnalyzer 패턴 적용** | START_HERE + NEXT_STEP slug = 적용, Phase/session bootstrap = 불필요 |
| **최종 효과** | 사람 조작 50% 감소, 프롬프트 74% 감소, 생성 대기 100% 제거 |

---

## 8. 다음 액션 (선택지)

```
A. 이 검토 기반으로 CC_ENV_SWITCH 패키지 전체 제작 (scripts/ + templates/ + docs)
B. cc-switch.ps1만 먼저 사전 제작 (가장 효과 큰 단일 항목)
C. 현재 문서를 최소 수정으로 분리만 (자동화는 나중에)
D. 검토 내용 조정 요청
```
