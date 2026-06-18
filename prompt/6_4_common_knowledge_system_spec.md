# 6.4 Common Knowledge System — 상세 설계 Spec

---

## 1. 목적

이 문서는 [`master/L1_AI_Automation_Roadmap_v0.38.md`](L1a/master/L1_AI_Automation_Roadmap_v0.38.md) 의 `6.4 Common Knowledge System` 항목에 대한 상세 설계를 정의한다.

Common Knowledge System은 5.1 Jira Feedback Loop, 5.2 Code Analyzer, 5.4 HLD↔Code Consistency Check, 5.5 RCA Knowledge Graph, 5.6 Onboarding Knowledge Pack이 공유하는 지식 축적 및 검색 기반이다.

---

## 2. 역할 정의

### 2.1 Skill vs Knowledge DB

```text
Skill
= AI가 defect를 판단할 때 따라야 하는 사고 방식과 검토 규칙
= Thinking Rule

Knowledge DB
= 실제 Resolve / Reject 경험 데이터
= Experience
```

Skill과 Knowledge DB는 역할이 다르다. Skill은 판단 기준이고, Knowledge DB는 판단에 참조할 경험 데이터다.

### 2.2 공통 참조 대상

```text
- 5.1 Jira Feedback Loop       ← Resolve/Reject 사례, Prevent Rule, False Positive Rule 저장
- 5.2 Code Analyzer            ← API Call Flow, 함수/클래스 구조 분석 결과 저장
- 5.4 HLD↔Code Consistency     ← Gap Pattern, HLD 누락 유형 저장
- 5.5 RCA Knowledge Graph      ← Issue, Root Cause, Fix, Jira, HLD, TC 관계 저장
- 5.6 Onboarding Knowledge Pack ← Domain Guide, API List, RCA Case 참조
```

---

## 3. 하위 구성

### 3.1 Knowledge Architecture

```text
l1_knowledge_system/
├── cases/
│   ├── resolved/          ← Resolve 사례 Markdown
│   └── rejected/          ← Reject(False Positive) 사례 Markdown
├── rules/
│   ├── active_prevent_rules.md    ← 현재 유효한 Prevent Rule
│   └── reject_patterns.md         ← False Positive 패턴 목록
├── db/
│   └── l1_knowledge.sqlite        ← Phase 2 이후 사용
├── scripts/
│   ├── add_case.py        ← 사례 추가
│   ├── build_index.py     ← 인덱스 생성 (Phase 2~)
│   └── search_case.py     ← 사례 검색
└── README.md
```

### 3.2 Resolve / Reject Case Management

**Resolve 사례 저장 형식**

```markdown
## Case ID: RESOLVE-YYYYMMDD-NNN
- Jira: [Jira Key]
- P4 Shelve: [Shelve Number]
- 파일/함수: [대상 파일, 함수명]
- 도메인: [도메인 태그]
- Defect Type: [결함 유형]
- Root Cause: [원인 요약]
- Fix 요약: [수정 내용 요약]
- Prevent Rule: [생성된 Prevent Rule]
- Review Comment: [주요 Review 의견]
- Date: [날짜]
```

**Reject 사례 저장 형식**

```markdown
## Case ID: REJECT-YYYYMMDD-NNN
- Jira: [Jira Key]
- P4 Shelve: [Shelve Number]
- 파일/함수: [대상 파일, 함수명]
- 도메인: [도메인 태그]
- Defect Type: [결함 유형]
- False Positive 이유: [과탐 판단 근거]
- False Positive Rule: [생성된 False Positive Rule]
- Review Comment: [주요 Review 의견]
- Date: [날짜]
```

### 3.3 Prevent Rule Management

Prevent Rule은 `rules/active_prevent_rules.md`에 도메인 태그 기준으로 관리한다.

```markdown
## Domain: ENDC
- Peer RAT 존재 시 AS/PS conflict를 RF API 호출 전에 확인한다.
- ENDC MCG/SCG 우선순위 조건을 switch decision 이전에 검증한다.

## Domain: Dual SIM
- ...
```

Rule은 버전을 관리하지 않고 현재 유효한 Rule만 유지한다. Rule 변경 이력은 사례 파일에서 추적한다.

### 3.4 Index-based Knowledge System

사례가 누적될수록 전체 Knowledge DB를 AI context에 주입하는 것은 비효율적이다. 아래 순서로 선별하여 주입한다.

```text
1. 변경 파일 / 함수 / 도메인 / Defect Type 추출
2. Metadata index 검색
3. 유사 Resolve case Top K 조회
4. 유사 Reject case Top K 조회
5. 관련 Prevent Rule 조회
6. 선별된 내용만 AI context에 주입
```

### 3.5 Long-term Knowledge Roadmap

상세는 아래 Phase 1~4 Roadmap 섹션 참조.

---

## 4. Phase 1~4 Roadmap

### Phase 1 (현재 구현 범위)

```text
Skill Update
+
Markdown 기록
```

목적:
- Resolve / Reject 결과를 Skill에 반영
- 별도 DB 없이 시작
- 운영 부담 최소화

구현 대상:
- `cases/resolved/`, `cases/rejected/` 폴더에 Markdown 저장
- `rules/active_prevent_rules.md`, `rules/reject_patterns.md` 수동 관리
- `scripts/add_case.py` — 사례 추가 CLI

### Phase 2

```text
Markdown
+
SQLite FTS5
```

목적:
- 사례 검색 가능
- file / function / domain / defect type 기반 조회
- 별도 서버 없이 로컬 운영

구현 대상:
- `db/l1_knowledge.sqlite` 생성
- `scripts/build_index.py` — Markdown → SQLite FTS5 인덱스 생성
- `scripts/search_case.py` — 키워드 기반 사례 검색

### Phase 3

```text
Markdown
+
SQLite Metadata Index
+
FAISS Vector Index
```

목적:
- 의미 기반 유사 사례 검색
- 수천~수만 건 이상 누적 대응

구현 대상:
- SQLite Metadata Index (파일/함수/도메인/Defect Type 기반)
- FAISS Vector Index (임베딩 기반 유사도 검색)

### Phase 4

```text
LlamaIndex
+
Graphiti
```

목적:
- RAG 기반 retrieval layer 구성
- temporal knowledge graph 확장
- Jira / Shelve / RCA / HLD / TC 관계 추적

---

## 5. 검토한 오픈소스 Reference

아래 도구들은 장기 확장 후보로 검토하였다.

```text
- SQLite / SQLite FTS5   ← Phase 2 핵심
- FAISS                  ← Phase 3 벡터 검색
- ChromaDB               ← Phase 3 대안
- Qdrant                 ← Phase 3 대안
- LlamaIndex             ← Phase 4 RAG layer
- Graphiti               ← Phase 4 temporal knowledge graph
```

초기 구현은 아래 조합으로 시작한다.

```text
Markdown
+
SQLite FTS5
+
Python CLI
```

---

## 6. Claude Code 구현 가능성

Phase 1은 Claude Code로 즉시 구현 가능하다.

```text
l1_knowledge_system/
├── cases/
│   ├── resolved/
│   └── rejected/
├── rules/
│   ├── active_prevent_rules.md
│   └── reject_patterns.md
├── db/
│   └── l1_knowledge.sqlite
├── scripts/
│   ├── add_case.py
│   ├── build_index.py
│   └── search_case.py
└── README.md
```

Phase 2 이상은 별도 topic에서 상세 설계한다.

---

## 7. 다음 단계

```text
- Phase 1 구현: l1_knowledge_system 패키지 기본 구조 생성 (별도 topic)
- Phase 2 설계: SQLite FTS5 스키마 정의 (별도 topic)
- 5.1 Jira Feedback Loop와 연동 기준 확정
```
