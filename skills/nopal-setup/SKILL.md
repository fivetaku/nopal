---
name: nopal-setup
description: gws CLI 설치 및 Google Workspace OAuth 인증 가이드. gws CLI가 설치되지 않았거나 인증이 필요할 때 커맨드에서 Read로 읽혀 참고 자료로 사용됩니다. "gws 설치", "인증 안내", "OAuth 설정", "setup" 같은 요청에 사용됩니다.
---

# Nopal Setup — gws CLI 설치 및 인증 가이드

> 이 문서는 커맨드(`commands/nopal.md`)에서 Read로 읽히는 **참고 자료**다.
> gws CLI가 미설치이거나 인증이 안 된 경우에만 이 문서를 참조한다.

---

## 1. gws CLI란?

gws(Google Workspace CLI)는 Google Workspace의 9개 서비스를 터미널에서 조작할 수 있는 공식 CLI 도구다.
Nopal은 이 gws CLI를 통해 Google Workspace와 상호작용한다.

**핵심 포인트:**
- gws CLI가 OAuth 토큰을 자체 관리한다.
- Nopal 플러그인은 토큰을 직접 다루지 않는다.
- API 키나 서비스 계정이 아닌 OAuth(사용자 인증) 방식이다.

---

## 2. 설치

### 전제 조건

- Node.js 18 이상
- npm 또는 yarn

### 설치 명령어

```bash
npm install -g @googleworkspace/cli
```

### 설치 확인

```bash
gws --version
```

버전 번호가 출력되면 설치 성공이다.

### 설치 실패 시

| 증상 | 해결 방법 |
|------|----------|
| `command not found: gws` | npm 글로벌 경로가 PATH에 없을 수 있다. `npm config get prefix` 확인 후 PATH에 추가. |
| permission 에러 | `sudo npm install -g @googleworkspace/cli` 또는 nvm 사용 권장. |
| Node.js 버전 에러 | `node --version`으로 확인. 18 미만이면 업그레이드 필요. |

**안내 규칙:**
- 기본적으로 `/nopal` 실행 시 `gws` 미설치면 설치를 자동으로 시도한다.
- 자동 설치가 실패하면 사용자에게 수동 설치 방법을 안내한다.

---

## 3. 초기 설정 (`gws auth setup`)

> `gws auth setup`은 인터랙티브 TUI다. Bash 도구로 실행할 수 없으며, 사용자에게 터미널에서 직접 실행하도록 안내해야 한다.

### 3-1. 실행

```bash
gws auth setup
```

### 3-2. 5단계 가이드

**Step 1/5: gcloud CLI 확인**
- 자동 감지된다. 미설치 시 https://cloud.google.com/sdk/docs/install 참고.

**Step 2/5: Google 계정 선택**
- 사용할 Google 계정을 선택한다.

**Step 3/5: GCP 프로젝트 생성**
- "Create new project"를 선택한다 (추천).
- 프로젝트 ID 규칙:
  - 소문자 영문으로 시작
  - 소문자, 숫자, 하이픈(`-`) 만 사용 가능
  - 6~30자
  - 전 세계에서 유일해야 함 (중복 시 에러)
- 추천 형식: `nopal-ws-XXXXXX` (예: `nopal-ws-830621`)
  - 뒤에 생년월일이나 랜덤 숫자 6자리를 붙이면 겹칠 확률이 낮다.

**Step 4/5: Workspace API 선택**
- Nopal에 필요한 9개를 모두 선택:
  - Google Drive, Google Sheets, Gmail, Google Calendar
  - Google Docs, Google Slides, Google Tasks, Google Chat, Google Meet
- 나머지(People, Vault, Forms, Keep 등)는 선택하지 않는다.
- 선택을 적게 할수록 스코프 수가 줄어 인증이 쉬워진다.

**Step 5/5: OAuth 인증 정보 (가장 중요)**

A. OAuth 동의 화면 설정:
   - 터미널에 표시된 Step A 링크를 열거나 직접 이동:
     https://console.cloud.google.com/apis/credentials/consent
   - User Type: "External" → 만들기
   - 앱 이름 입력 (예: nopal), 지원 이메일 선택, 나머지 빈칸 → 저장

B. 테스트 사용자 추가 (**필수! 안 하면 "액세스 차단됨" 403 에러 발생**):
   - 동의 화면 설정에서 "Test users" 탭 → "+ Add users"
   - 본인 Gmail 주소 입력 → 저장
   - 직접 이동: https://console.cloud.google.com/apis/credentials/consent/edit

C. OAuth 클라이언트 ID 생성:
   - 터미널에 표시된 Step B 링크를 열거나 직접 이동:
     https://console.cloud.google.com/apis/credentials
   - "+ 사용자 인증 정보 만들기" → "OAuth 클라이언트 ID"
   - 애플리케이션 유형: "데스크톱 앱" → 만들기
   - 생성된 Client ID와 Client Secret을 터미널에 붙여넣기

---

## 4. 로그인 (`gws auth login`)

setup 완료 후 로그인을 진행한다:

```bash
gws auth login
```

- 스코프 선택 화면에서 "Recommended"를 선택한다.
- URL이 표시되면 **직접 브라우저에 복사해서** 열기 (자동으로 안 열린다).
- "Google hasn't verified this app" 경고 → **고급** → **앱으로 이동(안전하지 않음)** 클릭.
- Google 계정으로 로그인하고 권한을 승인하면 "Authentication successful" 표시.

**로그인 후 반드시 실행:**
```bash
gws auth export --unmasked > ~/.config/gws/credentials.json
```
이 명령어로 plain credentials 파일을 생성해야 Claude Code에서 gws를 사용할 수 있다.
(암호화된 credentials.enc는 OS Keyring에 의존하여 Bash 도구에서 접근이 안 될 수 있다.)

### 인증 확인

```bash
gws auth status
```

정상이면 인증된 Google 계정 이메일과 활성화된 API 목록이 표시된다.

---

## 5. 인증 문제 해결

| 증상 | 원인 | 해결 방법 |
|------|------|----------|
| `Token expired` | OAuth 토큰 만료 | `gws auth login` 재실행 후 `gws auth export --unmasked > ~/.config/gws/credentials.json` |
| `Invalid credentials` | 토큰 파일 손상 | `gws auth logout` 후 재로그인 |
| `액세스 차단됨` / `Access blocked` (403) | Test user 미등록 | GCP 콘솔(https://console.cloud.google.com/apis/credentials/consent) → Test users → 본인 이메일 추가 |
| `invalid_scope` (400) | 스코프 과다 (25개 초과) | 불필요한 API 비활성화(https://console.cloud.google.com/apis/dashboard) 또는 새 프로젝트 생성 |
| `No credentials provided` (401) | 레거시 credentials.enc 충돌 | `gws auth export --unmasked > ~/.config/gws/credentials.json` 후 credentials.enc 삭제 |
| 브라우저가 안 열림 | URL 자동 오픈 미지원 | 터미널에 표시된 URL을 직접 복사해서 브라우저에 붙여넣기 |
| 조직 계정 제한 | Google Workspace 관리자 정책 | 관리자에게 gws CLI 앱 허용 요청 |
| 프로젝트 ID 중복 에러 | 이미 사용 중인 ID | 뒤에 랜덤 숫자를 변경하여 재시도 |

### 인증 초기화 (최후의 수단)

```bash
gws auth logout
gws auth setup
```

모든 토큰을 삭제하고 처음부터 다시 설정한다.

---

## 6. 설치 완료 후

설치와 인증이 모두 완료되면 `/nopal` 커맨드를 다시 실행한다.
커맨드가 자동으로 인증 상태를 감지하고 오케스트레이션 모드로 진입한다.

**확인 체크리스트:**
- [ ] `gws --version` → 버전 번호 출력
- [ ] `gws auth status` → 인증된 이메일 + API 목록 표시
- [ ] `/nopal` 실행 → 오케스트레이션 시작
