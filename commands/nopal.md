---
name: nopal
description: "Google Workspace 오케스트레이션 — 자연어로 9개 서비스를 자동 조합"
argument-hint: "[자연어 요청]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Agent
---

# /nopal Command

Google Workspace 9개 서비스(Gmail, Calendar, Drive, Sheets, Docs, Slides, Chat, Tasks, Meet)를 자연어로 자동 조합하여 실행한다.

사용자 요청: `$ARGUMENTS`

## 규칙

- 모든 질문은 반드시 AskUserQuestion 도구로 호출한다. 텍스트로 질문하지 않는다.
- AskUserQuestion을 allowed-tools에 절대 넣지 않는다.
- gws CLI를 통해서만 Google Workspace와 상호작용한다. 직접 HTTP 호출 금지.
- 읽기 전용 단순 조회(일정 확인, 메일 확인 등)는 확인 없이 바로 실행한다. 쓰기/변경 작업만 사용자 확인을 받는다.

## Step 0: 환경 확인 (자동)

gws CLI 설치 여부와 인증 상태를 자동으로 확인한다. 유저에게 묻지 않는다.

### 0-1. gws CLI 설치 확인

Bash로 `command -v gws`를 실행한다.

**미설치인 경우:**
Bash로 `npm install -g @googleworkspace/cli`를 실행하여 자동 설치한다. 설치 완료 후 `gws --version`으로 확인하고 0-2로 진행한다. 설치 실패 시 Read `${CLAUDE_PLUGIN_ROOT}/skills/nopal-setup/SKILL.md`를 읽고 수동 설치를 안내한다.

**설치된 경우:** 0-2로 진행.

### 0-2. 인증 상태 확인

Bash로 `gws auth status`를 실행한다.

**인증 안 된 경우:**

`gws auth setup`은 인터랙티브 TUI이므로 Bash 도구로 실행할 수 없다. 사용자에게 아래 안내를 텍스트로 보여주고 중단한다.

```
Google Workspace 초기 설정이 필요합니다.
터미널에서 아래 명령어를 실행해주세요:

  gws auth setup

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step 1/5: gcloud CLI 확인
  → 자동 감지됩니다.
  → 미설치 시: https://cloud.google.com/sdk/docs/install

Step 2/5: Google 계정 선택
  → 사용할 Google 계정을 선택하세요.

Step 3/5: GCP 프로젝트 생성
  → "Create new project"를 선택하세요 (추천).
  → 프로젝트 ID 규칙:
    • 소문자 영문으로 시작
    • 소문자, 숫자, 하이픈(-) 만 사용 가능
    • 6~30자
    • 전 세계에서 유일해야 함 (중복 불가)
  → 추천 형식: nopal-ws-XXXXXX (예: nopal-ws-830621)
    뒤에 생년월일이나 랜덤 숫자 6자리를 붙이면 겹칠 확률이 낮습니다.

Step 4/5: Workspace API 선택
  → Nopal에 필요한 9개를 모두 선택하세요:
    ✓ Google Drive
    ✓ Google Sheets
    ✓ Gmail
    ✓ Google Calendar
    ✓ Google Docs
    ✓ Google Slides
    ✓ Google Tasks
    ✓ Google Chat
    ✓ Google Meet
  → 나머지(People, Vault, Forms, Keep 등)는 선택하지 않아도 됩니다.
  → 선택을 적게 할수록 스코프 수가 줄어 인증이 쉬워집니다.

Step 5/5: OAuth 인증 정보 (가장 중요!)

  ┌─────────────────────────────────────┐
  │  A. OAuth 동의 화면 설정            │
  ├─────────────────────────────────────┤
  │  터미널에 표시된 Step A 링크를 열거나 │
  │  아래 링크로 직접 이동:              │
  │                                     │
  │  https://console.cloud.google.com/  │
  │  apis/credentials/consent           │
  │                                     │
  │  1. User Type: "External" → 만들기  │
  │  2. 앱 이름 입력 (예: nopal)        │
  │  3. 지원 이메일: 본인 이메일 선택    │
  │  4. 나머지는 빈칸 → "저장 후 계속"  │
  └─────────────────────────────────────┘

  ┌─────────────────────────────────────┐
  │  ⚠️  B. 테스트 사용자 추가 (필수!)  │
  ├─────────────────────────────────────┤
  │  이걸 안 하면 "액세스 차단됨"        │
  │  (403 access_denied) 에러가 납니다!  │
  │                                     │
  │  위 동의 화면 설정에서:              │
  │  "Test users" 탭 → "+ Add users"    │
  │  → 본인 Gmail 주소 입력 → 저장      │
  │                                     │
  │  또는 직접 이동:                     │
  │  https://console.cloud.google.com/  │
  │  apis/credentials/consent/edit      │
  │  → "테스트 사용자" 단계까지 진행     │
  └─────────────────────────────────────┘

  ┌─────────────────────────────────────┐
  │  C. OAuth 클라이언트 ID 생성        │
  ├─────────────────────────────────────┤
  │  터미널에 표시된 Step B 링크를 열거나 │
  │  아래 링크로 직접 이동:              │
  │                                     │
  │  https://console.cloud.google.com/  │
  │  apis/credentials                   │
  │                                     │
  │  1. "+ 사용자 인증 정보 만들기"      │
  │     → "OAuth 클라이언트 ID"          │
  │  2. 애플리케이션 유형: "데스크톱 앱"  │
  │  3. 이름: 아무거나 (예: nopal-cli)   │
  │  4. "만들기" 클릭                    │
  │  5. Client ID와 Client Secret 복사   │
  │  6. 터미널에 붙여넣기                │
  └─────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

setup이 끝나면 로그인을 진행합니다:

  gws auth login

URL이 표시되면 직접 브라우저에 복사해서 열어주세요 (자동으로 안 열립니다).
"Google hasn't verified this app" 경고가 나오면:
  → "고급" 클릭 → "앱이름(으)로 이동(안전하지 않음)" 클릭

Google 계정으로 로그인하고 권한을 승인하면 "Authentication successful" 이 표시됩니다.

⚠️ "invalid_scope" 에러가 나오면:
  → Step 4에서 불필요한 API를 너무 많이 선택한 경우입니다.
  → GCP 콘솔(https://console.cloud.google.com/apis/dashboard)에서
    불필요한 API를 비활성화하고 다시 로그인해보세요.
  → 또는 새 프로젝트를 만들어서 필요한 9개 API만 선택하세요.

완료되면 /nopal을 다시 실행해주세요.
```

여기서 중단한다. 사용자가 설정을 완료하고 다시 `/nopal`을 실행하면 `gws auth status`로 재확인 후 Step 1로 진행한다.

**인증 완료:** Step 1로 진행.

## Step 1: 요청 파싱

### 조건 A — `$ARGUMENTS`가 있는 경우

사용자의 자연어 요청을 그대로 오케스트레이션 입력으로 사용한다. Step 2로 바로 진행.

### 조건 B — `$ARGUMENTS`가 없는 경우

**EXECUTE:** AskUserQuestion 도구를 즉시 호출한다:

AskUserQuestion({
  "questions": [
    {
      "question": "무엇을 도와드릴까요?",
      "header": "Nopal",
      "multiSelect": false,
      "options": [
        {
          "label": "자유 요청 (추천)",
          "description": "하고 싶은 작업을 자유롭게 말해주세요. 예: '내일 오후 2시에 팀 회의 잡고 참석자에게 메일 보내줘'"
        },
        {
          "label": "오늘 일정 확인",
          "description": "오늘 하루의 Google Calendar 일정을 확인합니다."
        },
        {
          "label": "이메일 확인",
          "description": "Gmail에서 최근 읽지 않은 이메일을 확인합니다."
        },
        {
          "label": "사용법 안내",
          "description": "Nopal이 할 수 있는 일과 사용 방법을 안내합니다."
        }
      ]
    }
  ]
})

답변에 따라:
- "자유 요청" → 사용자가 입력한 텍스트를 오케스트레이션 입력으로 사용. Step 2로 진행.
- "오늘 일정 확인" → 요청을 "오늘 일정을 확인해줘"로 설정. Step 2로 진행.
- "이메일 확인" → 요청을 "읽지 않은 이메일을 확인해줘"로 설정. Step 2로 진행.
- "사용법 안내" → 아래 안내를 출력하고 종료:

```
Nopal은 Google Workspace 8개 서비스를 자연어로 자동 조합합니다.

지원 서비스: Gmail, Calendar, Drive, Sheets, Docs, Slides, Chat, Tasks, Meet

사용 예시:
  /nopal 내일 오전 10시에 팀 회의 잡아줘
  /nopal 지난주 매출 스프레드시트에서 합계 구해줘
  /nopal 회의록을 Google Docs로 만들고 참석자에게 공유해줘
  /nopal 읽지 않은 이메일 중 중요한 것만 요약해줘

여러 서비스를 조합하는 복잡한 요청도 가능합니다:
  /nopal 내일 회의 참석자 목록을 스프레드시트로 만들고 각자에게 메일로 안건 보내줘
```

## Step 2: 오케스트레이션 실행

Read `${CLAUDE_PLUGIN_ROOT}/skills/nopal-orchestrate/SKILL.md` 를 읽고, 해당 스킬의 워크플로우(5단계)에 따라 사용자 요청을 처리한다.

오케스트레이션 스킬에 전달할 정보:
- **사용자 요청**: Step 1에서 파싱된 자연어 요청
- **인증 상태**: Step 0에서 확인 완료

오케스트레이션 스킬이 완료되면 결과를 사용자에게 보여준다.
