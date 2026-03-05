---
name: nopal
description: "Google Workspace 오케스트레이션 — 자연어로 8개 서비스를 자동 조합"
argument-hint: "[자연어 요청]"
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
  - Agent
---

# /nopal Command

Google Workspace 8개 서비스(Gmail, Calendar, Drive, Sheets, Docs, Slides, Chat, Tasks)를 자연어로 자동 조합하여 실행한다.

사용자 요청: `$ARGUMENTS`

## 규칙

- 모든 질문은 반드시 AskUserQuestion 도구로 호출한다. 텍스트로 질문하지 않는다.
- AskUserQuestion을 allowed-tools에 절대 넣지 않는다.
- gws CLI를 통해서만 Google Workspace와 상호작용한다. 직접 HTTP 호출 금지.
- 실행 계획을 세운 뒤 반드시 사용자 확인을 받고 실행한다.

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

사용자에게 아래 안내를 보여준 후 터미널에서 직접 실행하도록 한다. `gws auth setup`은 인터랙티브 TUI이므로 Bash 도구로 실행할 수 없다.

```
Google Workspace 초기 설정이 필요합니다. 터미널에서 아래 명령어를 실행해주세요:

  gws auth setup

5단계를 순서대로 진행합니다:

  Step 1/5: gcloud CLI 확인 — 자동 감지됩니다
  Step 2/5: Google 계정 선택 — 사용할 Google 계정을 선택하세요
  Step 3/5: GCP 프로젝트 — "Create new project"를 선택하거나 기존 프로젝트를 선택하세요
  Step 4/5: Workspace API — 8개 서비스를 모두 선택하세요:
            Google Drive, Google Sheets, Gmail, Google Calendar,
            Google Docs, Google Slides, Google Tasks, Google Chat
  Step 5/5: OAuth 인증 정보 — 가장 중요한 단계입니다:
            1. 터미널에 표시된 Step A 링크를 열어 OAuth 동의 화면을 설정하세요
               → User Type: External → 저장
            2. Step B 링크를 열어 OAuth 클라이언트를 생성하세요
               → "사용자 인증 정보 만들기" → "OAuth 클라이언트 ID"
               → 애플리케이션 유형: 데스크톱 앱
               → 만들기
            3. 생성된 Client ID와 Client Secret을 터미널에 붙여넣으세요

setup이 끝나면 로그인을 진행합니다. 스코프를 8개 서비스로 제한해야 합니다:

  gws auth login --scopes drive,gmail,calendar,sheets,docs,slides,chat,tasks

URL이 표시되면 직접 브라우저에 복사해서 열어주세요 (자동으로 안 열립니다).
Google 계정으로 로그인하고 권한을 승인하면 터미널에 "Authentication successful" 이 표시됩니다.

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

지원 서비스: Gmail, Calendar, Drive, Sheets, Docs, Slides, Chat, Tasks

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
