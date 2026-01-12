# GSD (Get Shit Done) 완벽 한글 가이드

**Claude Code를 위한 메타-프롬프팅, 컨텍스트 엔지니어링, 스펙 기반 개발 시스템**

---

## 목차

1. [개요](#1-개요)
2. [설치 방법](#2-설치-방법)
3. [핵심 개념](#3-핵심-개념)
4. [프로젝트 구조](#4-프로젝트-구조)
5. [워크플로우 모드](#5-워크플로우-모드)
6. [명령어 상세 설명](#6-명령어-상세-설명)
7. [파일 템플릿 상세](#7-파일-템플릿-상세)
8. [실전 사용 예시](#8-실전-사용-예시)
9. [고급 기능](#9-고급-기능)
10. [문제 해결](#10-문제-해결)

---

## 1. 개요

### GSD란 무엇인가?

GSD(Get Shit Done)는 Claude Code를 사용하여 소프트웨어 프로젝트를 효율적으로 개발하기 위한 **메타-프롬프팅 시스템**입니다.

**핵심 철학:**
- 당신은 **아이디어를 가진 사람** (비전/제품 오너)
- Claude는 **실제 코드를 작성하는 빌더**
- 복잡한 엔터프라이즈 프로세스 없이 **빠르게 결과물을 만들어냄**

### 왜 GSD인가?

기존의 스펙 기반 개발 도구들(BMAD, Speckit 등)은 스프린트 세리머니, 스토리 포인트, 이해관계자 회의 등 **엔터프라이즈 복잡성**을 강요합니다.

GSD는 다릅니다:
- **단순한 워크플로우**: 몇 가지 명령어만으로 동작
- **숨겨진 복잡성**: 컨텍스트 엔지니어링, XML 프롬프트 포맷팅, 서브에이전트 오케스트레이션이 뒤에서 작동
- **검증 내장**: Claude가 작업을 완료하고 직접 검증

### GSD가 해결하는 문제

**바이브코딩의 문제점:**
1. 일관성 없는 코드 품질
2. 규모가 커지면 무너지는 구조
3. Claude의 컨텍스트 윈도우 소진으로 인한 품질 저하

**GSD의 해결책:**
1. **컨텍스트 엔지니어링**: Claude에게 필요한 모든 정보를 체계적으로 제공
2. **원자적 계획**: 2-3개 태스크씩 작은 단위로 분할
3. **서브에이전트 실행**: 각 계획을 새로운 컨텍스트에서 실행하여 품질 유지

---

## 2. 설치 방법

### 기본 설치

```bash
npx get-shit-done-cc
```

설치 후 `/gsd:help` 명령으로 설치 확인

### 설치 옵션

| 옵션 | 설명 | 사용처 |
|------|------|--------|
| `--global` 또는 `-g` | `~/.claude/`에 전역 설치 | 모든 프로젝트에서 사용 |
| `--local` 또는 `-l` | `./.claude/`에 로컬 설치 | 특정 프로젝트에서만 사용 |

### Docker/CI 환경 설치

```bash
# Docker 환경에서 경로 문제 해결
CLAUDE_CONFIG_DIR=/home/youruser/.claude npx get-shit-done-cc --global
```

### 권장 실행 모드

GSD는 자동화를 위해 설계되었으므로 권한 확인 모드 생략을 권장합니다:

```bash
claude --dangerously-skip-permissions
```

> **참고**: `date`, `git commit` 같은 명령을 50번씩 승인하는 것은 GSD의 목적에 맞지 않습니다.

### 세부 권한 설정 (대안)

프로젝트의 `.claude/settings.json`에 추가:

```json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)"
    ]
  }
}
```

---

## 3. 핵심 개념

### 3.1 컨텍스트 엔지니어링

Claude Code는 **적절한 컨텍스트가 주어지면** 매우 강력합니다. 대부분의 사람들은 이것을 제공하지 않습니다.

GSD가 관리하는 컨텍스트:

| 파일 | 역할 |
|------|------|
| `PROJECT.md` | 프로젝트 비전, 항상 로드됨 |
| `ROADMAP.md` | 어디로 가는지, 무엇이 완료되었는지 |
| `STATE.md` | 결정사항, 블로커, 위치 - 세션 간 메모리 |
| `PLAN.md` | XML 구조의 원자적 태스크, 검증 단계 포함 |
| `SUMMARY.md` | 무엇이 일어났는지, 무엇이 변경되었는지 |
| `ISSUES.md` | 세션 간 추적되는 지연된 개선사항 |

### 3.2 XML 프롬프트 포맷팅

모든 계획은 Claude에 최적화된 XML로 구조화됩니다:

```xml
<task type="auto">
  <name>로그인 엔드포인트 생성</name>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    JWT를 위해 jose 사용 (jsonwebtoken 아님 - CommonJS 이슈).
    users 테이블에 대해 인증 검증.
    성공 시 httpOnly 쿠키 반환.
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login이 200 + Set-Cookie 반환</verify>
  <done>유효한 인증정보는 쿠키 반환, 무효한 경우 401</done>
</task>
```

**구성 요소:**
- `name`: 태스크 이름
- `files`: 생성/수정할 파일 경로
- `action`: 구체적인 구현 지침
- `verify`: 완료 확인 방법
- `done`: 완료 기준

### 3.3 서브에이전트 실행

Claude의 컨텍스트 윈도우가 채워지면 품질이 저하됩니다:

| 컨텍스트 사용량 | 품질 |
|----------------|------|
| 0-30% | 최고 품질 |
| 30-50% | 좋은 품질 |
| 50-70% | 저하되는 품질 |
| 70%+ | 낮은 품질 |

**GSD의 해결책:**
- 각 계획은 최대 3개 태스크
- 각 계획은 새로운 서브에이전트에서 실행
- **200k 토큰을 순수하게 구현에만 사용**
- 품질 저하 없음

### 3.4 원자적 Git 커밋

각 태스크는 완료 직후 즉시 커밋됩니다:

```bash
abc123f docs(08-02): complete user registration plan
def456g feat(08-02): add email confirmation flow
hij789k feat(08-02): implement password hashing
lmn012o feat(08-02): create registration endpoint
```

**장점:**
- `git bisect`로 정확한 실패 태스크 찾기
- 각 태스크를 독립적으로 되돌릴 수 있음
- 향후 Claude 세션을 위한 명확한 히스토리
- AI 자동화 워크플로우에 최적화된 가시성

### 3.5 솔로 개발자 + Claude 모델

GSD는 **한 사람**(사용자)과 **한 구현자**(Claude)를 위해 설계되었습니다.

**포함하지 않는 것:**
- 팀 구조, RACI 매트릭스
- 이해관계자 관리
- 스프린트 세리머니
- 인간 개발 시간 추정 (시간, 일, 주)
- 변경 관리 프로세스
- 문서를 위한 문서

---

## 4. 프로젝트 구조

### 4.1 디렉토리 구조

```
.planning/
├── PROJECT.md            # 프로젝트 비전 및 요구사항
├── ROADMAP.md            # 현재 페이즈 분해
├── STATE.md              # 프로젝트 메모리 & 컨텍스트
├── ISSUES.md             # 지연된 개선사항 (필요시 생성)
├── config.json           # 워크플로우 모드 & 게이트
├── todos/                # 캡처된 아이디어 및 태스크
│   ├── pending/          # 작업 대기 중인 투두
│   └── done/             # 완료된 투두
├── codebase/             # 코드베이스 맵 (브라운필드 프로젝트)
│   ├── STACK.md          # 언어, 프레임워크, 의존성
│   ├── ARCHITECTURE.md   # 패턴, 레이어, 데이터 흐름
│   ├── STRUCTURE.md      # 디렉토리 레이아웃, 주요 파일
│   ├── CONVENTIONS.md    # 코딩 표준, 네이밍
│   ├── TESTING.md        # 테스트 설정, 패턴
│   ├── INTEGRATIONS.md   # 외부 서비스, API
│   └── CONCERNS.md       # 기술 부채, 알려진 이슈
└── phases/
    ├── 01-foundation/
    │   ├── 01-01-PLAN.md      # 첫 번째 계획
    │   └── 01-01-SUMMARY.md   # 실행 결과 요약
    └── 02-core-features/
        ├── 02-01-PLAN.md
        └── 02-01-SUMMARY.md
```

### 4.2 파일별 역할

| 파일 | 역할 | 업데이트 시점 |
|------|------|--------------|
| `PROJECT.md` | 프로젝트의 "무엇"과 "왜" | 주요 피봇이나 검증 후 |
| `ROADMAP.md` | 페이즈 목록과 진행 상황 | 계획/실행 후 |
| `STATE.md` | 현재 위치와 누적 컨텍스트 | 모든 실행 후 |
| `config.json` | 워크플로우 설정 | 모드 변경 시 |
| `ISSUES.md` | 지연된 개선사항 | 이슈 발견 시 |
| `PLAN.md` | 실행할 태스크들 | 페이즈 계획 시 |
| `SUMMARY.md` | 실행 결과 | 계획 실행 완료 후 |

---

## 5. 워크플로우 모드

### 5.1 Interactive 모드

**특징:**
- 각 주요 결정에서 확인 요청
- 체크포인트에서 승인 대기
- 더 많은 가이드 제공

**적합한 경우:**
- GSD를 처음 사용할 때
- 중요한 프로젝트에서 신중하게 진행하고 싶을 때
- 각 단계를 이해하며 진행하고 싶을 때

### 5.2 YOLO 모드

**특징:**
- 대부분의 결정을 자동 승인
- 확인 없이 계획 실행
- 중요한 체크포인트에서만 멈춤

**적합한 경우:**
- GSD에 익숙해진 후
- 빠르게 결과물을 얻고 싶을 때
- Claude를 완전히 신뢰할 때

### 5.3 모드 변경 방법

`.planning/config.json` 파일을 직접 수정:

```json
{
  "mode": "yolo",  // 또는 "interactive"
  "depth": "standard",
  "gates": {
    "execute_next_plan": true,
    "issues_review": true
  }
}
```

---

## 6. 명령어 상세 설명

### 6.1 프로젝트 초기화

#### `/gsd:new-project`

**목적:** 새 프로젝트를 심층적인 컨텍스트 수집을 통해 초기화

**실행 과정:**
1. 기존 프로젝트 확인 (이미 있으면 중단)
2. 기존 코드 감지 (브라운필드 감지)
3. 심층 질문을 통한 요구사항 수집
4. `PROJECT.md` 생성
5. 워크플로우 모드 선택
6. `config.json` 생성
7. Git 커밋

**생성 파일:**
- `.planning/PROJECT.md`
- `.planning/config.json`

**사용 예:**
```
/gsd:new-project
```

**질문 흐름:**
1. "무엇을 만들고 싶나요?" (자유 응답)
2. 언급한 내용에 대한 후속 질문 (선택지 제공)
3. "하나만 제대로 해야 한다면?" (핵심 가치)
4. "v1에서 명확히 제외할 것?" (범위 경계)
5. "하드 제약조건?" (기술, 시간, 예산 등)
6. "PROJECT.md 생성 준비됐나요?"

---

#### `/gsd:create-roadmap`

**목적:** 초기화된 프로젝트의 로드맵과 상태 추적 생성

**실행 과정:**
1. `PROJECT.md` 읽기
2. 페이즈 분해 (설정된 depth에 따라)
3. `ROADMAP.md` 생성
4. `STATE.md` 생성
5. 페이즈 디렉토리 생성
6. Git 커밋

**생성 파일:**
- `.planning/ROADMAP.md`
- `.planning/STATE.md`
- `.planning/phases/` 디렉토리들

**Depth 설정에 따른 차이:**

| Depth | 페이즈 수 | 페이즈당 계획 수 |
|-------|----------|-----------------|
| Quick | 3-5 | 1-3 |
| Standard | 5-8 | 3-5 |
| Comprehensive | 8-12 | 5-10 |

---

#### `/gsd:map-codebase`

**목적:** 기존 코드베이스를 분석하여 문서화 (브라운필드 프로젝트용)

**실행 과정:**
1. 병렬 Explore 에이전트 생성
2. 코드베이스 분석
3. 7개 문서 생성

**생성 파일:**

| 파일 | 내용 |
|------|------|
| `STACK.md` | 언어, 프레임워크, 의존성 |
| `ARCHITECTURE.md` | 패턴, 레이어, 데이터 흐름 |
| `STRUCTURE.md` | 디렉토리 구조, 주요 파일 위치 |
| `CONVENTIONS.md` | 코딩 스타일, 네이밍 규칙 |
| `TESTING.md` | 테스트 프레임워크, 패턴 |
| `INTEGRATIONS.md` | 외부 서비스, API 연동 |
| `CONCERNS.md` | 기술 부채, 알려진 이슈 |

**사용 시점:**
- 기존 코드베이스에 새 기능을 추가할 때
- `/gsd:new-project` 전에 실행

---

### 6.2 페이즈 계획

#### `/gsd:discuss-phase <number>`

**목적:** 페이즈 계획 전에 비전을 명확히 함

**실행 과정:**
1. 페이즈에 대한 사용자 비전 수집
2. 필수 요소와 경계 파악
3. `CONTEXT.md` 생성

**사용 예:**
```
/gsd:discuss-phase 2
```

**적합한 경우:**
- 페이즈가 어떻게 보이고 느껴져야 하는지 아이디어가 있을 때
- 계획 전에 방향을 명확히 하고 싶을 때

---

#### `/gsd:research-phase <number>`

**목적:** 틈새/복잡한 도메인에 대한 종합적인 생태계 연구

**실행 과정:**
1. 표준 스택과 아키텍처 패턴 조사
2. 함정과 모범 사례 파악
3. `RESEARCH.md` 생성

**사용 예:**
```
/gsd:research-phase 3
```

**적합한 경우:**
- 3D, 게임, 오디오, 셰이더, ML 등 전문 도메인
- "전문가는 어떻게 이것을 만드는가?" 지식이 필요할 때

---

#### `/gsd:list-phase-assumptions <number>`

**목적:** Claude가 계획하려는 접근 방식을 미리 확인

**실행 과정:**
1. Claude의 의도된 접근 방식 표시
2. 사용자가 방향 수정 가능
3. 파일 생성 없음 (대화식 출력만)

**사용 예:**
```
/gsd:list-phase-assumptions 3
```

**적합한 경우:**
- Claude가 비전을 잘못 이해했을 수 있을 때
- 계획 전에 방향을 확인하고 싶을 때

---

#### `/gsd:plan-phase <number>`

**목적:** 특정 페이즈에 대한 상세 실행 계획 생성

**실행 과정:**
1. `PROJECT.md`, `ROADMAP.md`, `STATE.md` 읽기
2. CONTEXT.md나 RESEARCH.md 있으면 참조
3. 구체적이고 실행 가능한 태스크 분해
4. 검증 기준과 성공 척도 포함
5. `XX-YY-PLAN.md` 생성
6. Git 커밋

**생성 파일:**
- `.planning/phases/XX-phase-name/XX-YY-PLAN.md`

**사용 예:**
```
/gsd:plan-phase 1
```

**결과 예:**
```
.planning/phases/01-foundation/01-01-PLAN.md
```

**계획 구조:**
```xml
<objective>
  무엇과 왜
</objective>

<context>
  @참조할_파일들
</context>

<tasks>
  <task type="auto">
    <name>태스크 1</name>
    <files>파일 경로</files>
    <action>할 일</action>
    <verify>검증 방법</verify>
    <done>완료 기준</done>
  </task>
</tasks>

<verification>
  전체 검증 체크리스트
</verification>

<success_criteria>
  성공 기준
</success_criteria>
```

---

### 6.3 실행

#### `/gsd:execute-plan <path>`

**목적:** PLAN.md 파일을 직접 실행

**실행 과정:**
1. 프로젝트 상태 로드
2. 계획 파싱 및 실행 전략 결정
3. 태스크 순차 실행
4. 각 태스크 완료 후 즉시 커밋
5. `SUMMARY.md` 생성
6. `STATE.md` 업데이트
7. Git 메타데이터 커밋

**사용 예:**
```
/gsd:execute-plan .planning/phases/01-foundation/01-01-PLAN.md
```

**실행 전략:**

| 전략 | 조건 | 설명 |
|------|------|------|
| A: 완전 자율 | 체크포인트 없음 | 전체 계획을 서브에이전트가 실행 |
| B: 분할 실행 | 검증 체크포인트만 | 세그먼트별로 실행, 체크포인트에서 일시정지 |
| C: 결정 의존 | 결정 체크포인트 | 메인 컨텍스트에서 순차 실행 |

**일탈 규칙:**
실행 중 계획에 없는 작업을 발견하면:

| 규칙 | 트리거 | 행동 |
|------|--------|------|
| 1. 버그 자동 수정 | 코드가 의도대로 작동하지 않음 | 즉시 수정, Summary에 기록 |
| 2. 필수 기능 추가 | 정확성/보안을 위해 필요 | 즉시 추가, Summary에 기록 |
| 3. 블로커 수정 | 진행 불가 | 즉시 수정, Summary에 기록 |
| 4. 아키텍처 변경 | 구조적 수정 필요 | **멈추고 사용자에게 질문** |
| 5. 개선사항 기록 | 좋지만 지금 필수 아님 | ISSUES.md에 기록, 계속 진행 |

---

### 6.4 로드맵 관리

#### `/gsd:add-phase <description>`

**목적:** 현재 마일스톤 끝에 새 페이즈 추가

**사용 예:**
```
/gsd:add-phase "관리자 대시보드 추가"
```

---

#### `/gsd:insert-phase <after> <description>`

**목적:** 기존 페이즈 사이에 긴급 작업을 소수점 페이즈로 삽입

**사용 예:**
```
/gsd:insert-phase 7 "중요 인증 버그 수정"
```

**결과:** Phase 7.1 생성

**사용 시점:**
- 마일스톤 중간에 긴급 수정이 필요할 때
- 발견된 작업이 특정 페이즈 후에 바로 수행되어야 할 때

---

#### `/gsd:remove-phase <number>`

**목적:** 미래 페이즈를 제거하고 후속 페이즈 번호 재조정

**사용 예:**
```
/gsd:remove-phase 17
```

**결과:** Phase 17 삭제, Phase 18-20이 17-19로 변경

**주의:** 시작되지 않은 미래 페이즈에만 작동

---

### 6.5 마일스톤 관리

#### `/gsd:discuss-milestone`

**목적:** 다음 마일스톤에서 무엇을 빌드할지 파악

**실행 과정:**
1. 이전 마일스톤에서 출시된 것 검토
2. 추가, 개선, 수정할 기능 식별
3. 준비되면 `/gsd:new-milestone`로 라우팅

---

#### `/gsd:new-milestone <name>`

**목적:** 기존 프로젝트에 새 마일스톤과 페이즈 생성

**사용 예:**
```
/gsd:new-milestone "v2.0 기능들"
```

**실행 과정:**
1. ROADMAP.md에 마일스톤 섹션 추가
2. 페이즈 디렉토리 생성
3. STATE.md 업데이트

---

#### `/gsd:complete-milestone <version>`

**목적:** 완료된 마일스톤을 아카이브하고 다음 버전 준비

**사용 예:**
```
/gsd:complete-milestone 1.0.0
```

**실행 과정:**
1. MILESTONES.md 항목 생성 (통계 포함)
2. milestones/ 디렉토리에 전체 상세 아카이브
3. 릴리스용 Git 태그 생성
4. 다음 버전을 위한 작업공간 준비

---

### 6.6 진행 추적

#### `/gsd:progress`

**목적:** 프로젝트 상태 확인 및 다음 행동으로 지능적 라우팅

**표시 내용:**
- 시각적 진행 바와 완료 퍼센트
- SUMMARY 파일에서 최근 작업 요약
- 현재 위치와 다음 단계
- 주요 결정사항과 열린 이슈

**다음 단계 제안:**
- 다음 계획 실행 또는 없으면 생성 제안
- 100% 마일스톤 완료 감지

**사용 예:**
```
/gsd:progress
```

---

### 6.7 세션 관리

#### `/gsd:resume-work`

**목적:** 이전 세션에서 전체 컨텍스트를 복원하여 작업 재개

**표시 내용:**
- STATE.md에서 프로젝트 컨텍스트 읽기
- 현재 위치와 최근 진행 상황 표시
- 프로젝트 상태에 따른 다음 행동 제안

---

#### `/gsd:pause-work`

**목적:** 페이즈 중간에 작업을 멈출 때 컨텍스트 핸드오프 생성

**실행 과정:**
1. 현재 상태로 `.continue-here` 파일 생성
2. STATE.md 세션 연속성 섹션 업데이트
3. 진행 중인 작업 컨텍스트 캡처

---

### 6.8 이슈 및 투두 관리

#### `/gsd:consider-issues`

**목적:** 코드베이스 컨텍스트와 함께 지연된 이슈 검토

**실행 과정:**
1. 모든 열린 이슈를 현재 코드베이스 상태와 분석
2. 해결된 이슈 식별 (닫을 수 있음)
3. 긴급한 이슈 식별 (지금 처리해야 함)
4. 다가오는 페이즈에 자연스럽게 맞는 것들 식별
5. 일괄 작업 제안 (닫기, 페이즈 삽입, 계획시 참고)

---

#### `/gsd:add-todo [description]`

**목적:** 현재 대화 컨텍스트에서 아이디어나 태스크를 투두로 캡처

**사용 예:**
```
/gsd:add-todo                    # 대화에서 추론
/gsd:add-todo 인증 토큰 갱신 추가   # 명시적 설명으로
```

**실행 과정:**
1. 대화에서 컨텍스트 추출 (또는 제공된 설명 사용)
2. `.planning/todos/pending/`에 구조화된 투두 파일 생성
3. 그룹화를 위해 파일 경로에서 영역 추론
4. 생성 전 중복 확인
5. STATE.md 투두 카운트 업데이트

---

#### `/gsd:check-todos [area]`

**목적:** 대기 중인 투두 목록을 보고 하나를 선택하여 작업

**사용 예:**
```
/gsd:check-todos           # 모든 투두 나열
/gsd:check-todos api       # 영역별 필터링
```

**실행 과정:**
1. 제목, 영역, 경과 시간과 함께 모든 대기 중 투두 나열
2. 선택된 투두의 전체 컨텍스트 로드
3. 적절한 행동으로 라우팅 (지금 작업, 페이즈에 추가, 브레인스토밍)
4. 작업 시작 시 투두를 done/으로 이동

---

### 6.9 검증

#### `/gsd:verify-work`

**목적:** 최근 빌드된 기능의 수동 사용자 수락 테스트 가이드

**실행 과정:**
1. 페이즈 또는 계획 범위 결정
2. SUMMARY.md에서 빌드된 것 요약
3. 테스트 시나리오 제시
4. 사용자 피드백 수집
5. 이슈가 발견되면 `/gsd:plan-fix`로 라우팅

---

#### `/gsd:plan-fix [plan]`

**목적:** verify-work에서 발견된 UAT 이슈에 대한 수정 계획

**실행 과정:**
1. UAT 이슈 분석
2. 수정 계획 생성
3. 실행 가능한 계획 파일 생성

---

### 6.10 유틸리티

#### `/gsd:help`

**목적:** 명령어 참조 표시

---

## 7. 파일 템플릿 상세

### 7.1 PROJECT.md

프로젝트의 "무엇"과 "왜"를 정의합니다.

```markdown
# [프로젝트 이름]

## What This Is

[현재 정확한 설명 — 2-3 문장. 이 제품이 무엇을 하고 누구를 위한 것인가?]

## Core Value

[가장 중요한 하나. 다른 모든 것이 실패해도 이것은 작동해야 함.
트레이드오프가 발생할 때 우선순위를 정하는 하나의 문장.]

## Requirements

### Validated
<!-- 출시되고 가치가 확인된 것들 -->
(아직 없음 — 출시하여 검증)

### Active
<!-- 현재 범위. 이것을 향해 빌드 중. -->
- [ ] [요구사항 1]
- [ ] [요구사항 2]

### Out of Scope
<!-- 명시적 경계. 다시 추가하는 것을 방지하기 위한 이유 포함. -->
- [제외 1] — [이유]

## Context

[구현에 정보를 제공하는 배경 정보]

## Constraints

- **[유형]**: [무엇] — [왜]

## Key Decisions

| 결정 | 근거 | 결과 |
|------|------|------|
| [선택] | [왜] | [✓ 좋음 / ⚠️ 재검토 / — 보류] |

---
*마지막 업데이트: [날짜] [트리거] 후*
```

**진화 규칙:**
- 페이즈 전환 후: 무효화된 요구사항 → Out of Scope, 검증된 것 → Validated
- 마일스톤 후: 모든 섹션 전체 검토

---

### 7.2 ROADMAP.md

프로젝트의 페이즈 분해와 진행 상황을 추적합니다.

```markdown
# Roadmap: [프로젝트 이름]

## Overview

[시작부터 끝까지의 여정을 설명하는 한 단락]

## Phases

**페이즈 번호:**
- 정수 페이즈 (1, 2, 3): 계획된 마일스톤 작업
- 소수점 페이즈 (2.1, 2.2): 긴급 삽입 (INSERTED 표시)

- [ ] **Phase 1: [이름]** - [한 줄 설명]
- [ ] **Phase 2: [이름]** - [한 줄 설명]

## Phase Details

### Phase 1: [이름]
**Goal**: [이 페이즈가 전달하는 것]
**Depends on**: Nothing (첫 페이즈)
**Research**: Unlikely (확립된 패턴)
**Plans**: [계획 수, 예: "3 plans" 또는 "TBD"]

Plans:
- [ ] 01-01: [첫 번째 계획 간략 설명]
- [ ] 01-02: [두 번째 계획 간략 설명]

## Progress

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. [이름] | 0/3 | Not started | - |
```

**상태 값:**
- `Not started` — 시작 안 함
- `In progress` — 현재 작업 중
- `Complete` — 완료 (완료 날짜 추가)
- `Deferred` — 나중으로 미룸 (이유와 함께)

---

### 7.3 STATE.md

프로젝트의 단기 메모리로, 모든 페이즈와 세션에 걸쳐 유지됩니다.

```markdown
# Project State

## Project Reference

See: .planning/PROJECT.md (updated [날짜])

**Core value:** [PROJECT.md Core Value 섹션에서 한 줄]
**Current focus:** [현재 페이즈 이름]

## Current Position

Phase: [X] of [Y] ([페이즈 이름])
Plan: [A] of [B] in current phase
Status: [Ready to plan / Planning / Ready to execute / In progress / Phase complete]
Last activity: [YYYY-MM-DD] — [무슨 일이 있었는지]

Progress: [░░░░░░░░░░] 0%

## Performance Metrics

**Velocity:**
- Total plans completed: [N]
- Average duration: [X] min
- Total execution time: [X.X] hours

## Accumulated Context

### Decisions
PROJECT.md Key Decisions 테이블에 결정사항 기록.

### Deferred Issues
[ISSUES.md에서 — 발생 페이즈와 함께 열린 항목 목록]

### Pending Todos
[.planning/todos/pending/에서 — 세션 중 캡처된 아이디어]

### Blockers/Concerns
[향후 작업에 영향을 미치는 이슈]

## Session Continuity

Last session: [YYYY-MM-DD HH:MM]
Stopped at: [마지막 완료된 행동 설명]
Resume file: [.continue-here*.md 경로 또는 "None"]
```

**크기 제약:** 100줄 이하 유지

---

### 7.4 PLAN.md (계획 파일)

Claude가 직접 실행할 수 있는 프롬프트입니다.

```markdown
---
phase: XX-name
type: execute
---

<objective>
[무엇과 왜]
Purpose: [...]
Output: [...]
</objective>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@관련/소스/파일.ts
</context>

<tasks>
<task type="auto">
  <name>Task N: [이름]</name>
  <files>[경로들]</files>
  <action>[할 일, 피할 것과 이유]</action>
  <verify>[명령/확인]</verify>
  <done>[기준]</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[Claude가 자동화한 것]</what-built>
  <how-to-verify>[번호 매긴 검증 단계]</how-to-verify>
  <resume-signal>[계속 방법 - "approved" 또는 이슈 설명]</resume-signal>
</task>
</tasks>

<verification>
[전체 페이즈 체크]
</verification>

<success_criteria>
[측정 가능한 완료]
</success_criteria>

<output>
[SUMMARY.md 명세]
</output>
```

**태스크 유형:**

| 유형 | 설명 | 사용 시점 |
|------|------|----------|
| `auto` | Claude가 자율적으로 실행 | 대부분의 코딩 작업 |
| `checkpoint:human-verify` | Claude 작업을 인간이 검증 | UI/UX 확인, 시각적 체크 |
| `checkpoint:decision` | 인간이 구현 선택 | 기술 선택, 아키텍처 결정 |
| `checkpoint:human-action` | 인간만 할 수 있는 행동 | 이메일 링크, 2FA 코드 (드묾) |

---

### 7.5 SUMMARY.md

계획 실행 결과를 문서화합니다.

```markdown
---
phase: XX-name
plan: XX-YY
subsystem: [분류]
tags: [기술 키워드]
duration: [소요 시간]
completed: [완료 날짜]
---

# Phase X Plan Y: [이름] Summary

**[실질적인 한 줄 요약]**

## Accomplishments

- [달성 1]
- [달성 2]

## Files Created/Modified

- `path/to/file.ts` — [변경 내용]

## Decisions Made

- [결정 1]: [근거]

## Deviations from Plan

### Auto-fixed Issues
**1. [Rule 1 - Bug] [설명]**
- **Found during:** Task X
- **Issue:** [이슈]
- **Fix:** [수정]
- **Commit:** abc123f

## Issues Encountered

None 또는 [이슈 목록]

## Next Phase Readiness

- [다음 페이즈를 위한 준비 상태]
```

---

## 8. 실전 사용 예시

### 8.1 새 프로젝트 시작하기

```bash
# 1. 프로젝트 초기화
/gsd:new-project

# 2. 로드맵 생성
/gsd:create-roadmap

# 3. 첫 번째 페이즈 계획
/gsd:plan-phase 1

# 4. 계획 실행
/gsd:execute-plan .planning/phases/01-foundation/01-01-PLAN.md
```

### 8.2 휴식 후 작업 재개

```bash
# 현재 상태 확인 및 다음 단계 안내
/gsd:progress
```

### 8.3 긴급 작업 추가

```bash
# 페이즈 5와 6 사이에 긴급 보안 수정 삽입
/gsd:insert-phase 5 "중요 보안 수정"

# 새 페이즈 계획
/gsd:plan-phase 5.1

# 실행
/gsd:execute-plan .planning/phases/05.1-critical-security-fix/05.1-01-PLAN.md
```

### 8.4 마일스톤 완료

```bash
# 마일스톤 완료 및 아카이브
/gsd:complete-milestone 1.0.0

# 다음 마일스톤 논의
/gsd:discuss-milestone

# 새 마일스톤 생성
/gsd:new-milestone "v2.0 기능들"
```

### 8.5 기존 프로젝트에 기능 추가

```bash
# 1. 먼저 코드베이스 매핑 (첫 번째만)
/gsd:map-codebase

# 2. 프로젝트 초기화 (기존 코드 인식)
/gsd:new-project

# 3. 이후는 새 프로젝트와 동일
/gsd:create-roadmap
/gsd:plan-phase 1
/gsd:execute-plan ...
```

### 8.6 아이디어 캡처 및 작업

```bash
# 대화 중 아이디어 캡처
/gsd:add-todo

# 또는 명시적으로
/gsd:add-todo 모달 z-index 수정

# 나중에 투두 검토 및 작업
/gsd:check-todos

# 특정 영역 필터링
/gsd:check-todos api
```

---

## 9. 고급 기능

### 9.1 TDD (테스트 주도 개발) 계획

비즈니스 로직, API 엔드포인트 등 테스트 가능한 동작이 있는 기능에 적합합니다.

**TDD 후보:**
- 정의된 입력/출력이 있는 비즈니스 로직
- API 엔드포인트 및 핸들러
- 데이터 변환 및 파싱
- 유효성 검사 규칙
- 상태 머신 및 워크플로우

**TDD를 건너뛸 경우:**
- UI 레이아웃 및 스타일링
- 탐색적 프로토타이핑
- 일회성 스크립트
- 구성 변경

**결정 휴리스틱:**
`expect(fn(input)).toBe(output)`를 fn 작성 전에 쓸 수 있나요?
- 예 → TDD 계획 생성
- 아니오 → 표준 계획, 필요시 이후 테스트 추가

**TDD 커밋 패턴:**
1. `test({phase}-{plan}): add failing test for X` (RED)
2. `feat({phase}-{plan}): implement X` (GREEN)
3. `refactor({phase}-{plan}): clean up X` (REFACTOR, 선택)

---

### 9.2 서브에이전트 재개

실행이 중단된 경우 서브에이전트를 재개할 수 있습니다.

```bash
/gsd:resume-task [agent-id]
```

**추적 파일:**
- `.planning/current-agent-id.txt` — 현재 실행 중인 에이전트 ID
- `.planning/agent-history.json` — 에이전트 실행 기록

---

### 9.3 인증 게이트

실행 중 인증 오류 발생 시 자동 처리:

1. 인증 게이트 인식 (버그 아님, 인증 필요)
2. 현재 태스크 실행 중지
3. 동적 체크포인트 생성
4. 정확한 인증 단계 제공
5. 사용자 인증 대기
6. 인증 작동 확인
7. 원래 태스크 재시도
8. 정상 계속

---

### 9.4 코드베이스 맵 업데이트

계획 실행 후 구조적 변경이 있으면 코드베이스 맵이 자동 업데이트됩니다:

| 감지된 변경 | 업데이트 행동 |
|------------|--------------|
| src/에 새 디렉토리 | STRUCTURE.md: 디렉토리 레이아웃에 추가 |
| package.json deps 변경 | STACK.md: 의존성 목록 추가/제거 |
| 새 파일 패턴 (예: 첫 .test.ts) | CONVENTIONS.md: 새 패턴 기록 |
| 새 외부 API 클라이언트 | INTEGRATIONS.md: 서비스 항목 추가 |

---

## 10. 문제 해결

### 10.1 설치 후 명령어를 찾을 수 없음

**해결책:**
1. Claude Code 재시작하여 슬래시 명령어 다시 로드
2. 파일 존재 확인:
   - 전역: `~/.claude/commands/gsd/`
   - 로컬: `./.claude/commands/gsd/`

### 10.2 명령어가 예상대로 작동하지 않음

**해결책:**
1. `/gsd:help` 실행하여 설치 확인
2. `npx get-shit-done-cc` 재실행하여 재설치

### 10.3 최신 버전으로 업데이트

```bash
npx get-shit-done-cc@latest
```

### 10.4 Docker 또는 컨테이너 환경 문제

틸드 경로(`~/.claude/...`)로 파일 읽기 실패 시:

```bash
CLAUDE_CONFIG_DIR=/home/youruser/.claude npx get-shit-done-cc --global
```

컨테이너에서 올바르게 확장되지 않을 수 있는 `~` 대신 절대 경로 사용을 보장합니다.

### 10.5 Git 커밋 실패

**가능한 원인:**
- Git이 초기화되지 않음
- 커밋할 변경사항 없음
- Git 사용자 설정 안 됨

**해결책:**
```bash
# Git 초기화 확인
git status

# 사용자 설정 (필요시)
git config user.name "Your Name"
git config user.email "your@email.com"
```

### 10.6 계획 실행 중 품질 저하

**가능한 원인:**
- 계획에 태스크가 너무 많음
- 컨텍스트 윈도우 소진

**해결책:**
- 계획당 2-3개 태스크로 분할
- `/clear` 명령으로 컨텍스트 정리 후 재시작
- `depth` 설정을 "comprehensive"로 변경하여 더 작은 계획 생성

---

## 부록: 명령어 빠른 참조

| 명령어 | 목적 |
|--------|------|
| `/gsd:new-project` | 새 프로젝트 초기화, PROJECT.md 생성 |
| `/gsd:create-roadmap` | 로드맵 및 상태 추적 생성 |
| `/gsd:map-codebase` | 기존 코드베이스 분석 (브라운필드용) |
| `/gsd:plan-phase [N]` | 페이즈에 대한 태스크 계획 생성 |
| `/gsd:execute-plan [path]` | 서브에이전트를 통해 계획 실행 |
| `/gsd:progress` | 현재 위치와 다음 단계 확인 |
| `/gsd:verify-work [N]` | 페이즈 또는 계획의 사용자 수락 테스트 |
| `/gsd:plan-fix [plan]` | verify-work에서 발견된 이슈 수정 계획 |
| `/gsd:complete-milestone` | 마일스톤 출시, 다음 버전 준비 |
| `/gsd:discuss-milestone` | 다음 마일스톤 컨텍스트 수집 |
| `/gsd:new-milestone [name]` | 새 마일스톤과 페이즈 생성 |
| `/gsd:add-phase` | 로드맵에 페이즈 추가 |
| `/gsd:insert-phase [N]` | 긴급 작업 삽입 |
| `/gsd:remove-phase [N]` | 미래 페이즈 제거, 후속 번호 재조정 |
| `/gsd:discuss-phase [N]` | 계획 전 컨텍스트 수집 |
| `/gsd:research-phase [N]` | 틈새 도메인 심층 생태계 연구 |
| `/gsd:list-phase-assumptions [N]` | Claude의 의도 확인 |
| `/gsd:pause-work` | 페이즈 중간 중단 시 핸드오프 파일 생성 |
| `/gsd:resume-work` | 마지막 세션에서 복원 |
| `/gsd:resume-task [id]` | 중단된 서브에이전트 실행 재개 |
| `/gsd:consider-issues` | 지연된 이슈 검토, 해결된 것 닫기 |
| `/gsd:add-todo [desc]` | 대화에서 아이디어/태스크 캡처 |
| `/gsd:check-todos [area]` | 대기 중 투두 나열, 작업할 것 선택 |
| `/gsd:help` | 모든 명령어 및 사용 가이드 표시 |

---

## 마무리

GSD는 **Claude Code를 강력하게, 그리고 신뢰할 수 있게** 만들어줍니다.

복잡한 엔터프라이즈 프로세스 없이, 아이디어를 설명하고 Claude가 올바르게 빌드하도록 하세요.

**핵심 원칙을 기억하세요:**
- 초기화에 시간을 투자하세요 (여기서의 깊은 질문 = 더 나은 결과)
- 계획을 작게 유지하세요 (2-3 태스크)
- Claude를 신뢰하되, 검증하세요
- 빠르게 출시하고, 배우고, 반복하세요

---

*이 문서는 GSD v1.3.34를 기반으로 작성되었습니다.*
