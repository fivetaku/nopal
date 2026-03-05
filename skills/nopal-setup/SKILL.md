---
name: nopal-setup
description: gws CLI 설치 및 Google Workspace OAuth 인증 가이드. gws CLI가 설치되지 않았거나 인증이 필요할 때 커맨드에서 Read로 읽혀 참고 자료로 사용됩니다.
---

# Nopal Setup — gws CLI 설치 및 인증 가이드

> 이 문서는 커맨드(`commands/nopal.md`)에서 Read로 읽히는 **참고 자료**다.
> gws CLI가 미설치이거나 인증이 안 된 경우에만 이 문서를 참조한다.

---

## 1. gws CLI란?

gws(Google Workspace CLI)는 Google Workspace의 8개 서비스를 터미널에서 조작할 수 있는 공식 CLI 도구다.
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

## 3. OAuth 인증

### 인증 실행

```bash
gws auth login
```

실행하면:
1. 브라우저가 자동으로 열린다.
2. Google 계정 선택 화면이 나타난다.
3. Google Workspace 권한을 승인한다.
4. 터미널에 "Authentication successful" 메시지가 표시된다.

### 인증 확인

```bash
gws auth status
```

정상이면 인증된 Google 계정 이메일이 표시된다.

### 인증에 필요한 권한 (scope)

gws CLI가 요청하는 주요 권한:
- Gmail: 이메일 읽기/보내기/관리
- Calendar: 일정 읽기/생성/수정
- Drive: 파일 읽기/업로드/공유
- Sheets: 스프레드시트 읽기/쓰기
- Docs: 문서 읽기/쓰기
- Slides: 프레젠테이션 읽기/쓰기
- Chat: 메시지 보내기/읽기
- Tasks: 할일 읽기/생성/수정

---

## 4. 인증 문제 해결

| 증상 | 원인 | 해결 방법 |
|------|------|----------|
| `Token expired` | OAuth 토큰 만료 | `gws auth login` 재실행 |
| `Invalid credentials` | 토큰 파일 손상 | `gws auth logout` 후 `gws auth login` 재실행 |
| 브라우저가 안 열림 | 원격 서버/headless 환경 | `gws auth login --no-browser` 후 출력된 URL을 직접 브라우저에 붙여넣기 |
| 권한 거부 에러 | 필요한 scope 미승인 | `gws auth login`으로 재인증하면서 모든 권한 승인 |
| 조직 계정 제한 | Google Workspace 관리자 정책 | 관리자에게 gws CLI 앱 허용 요청 |

### 인증 초기화 (최후의 수단)

```bash
gws auth logout
gws auth login
```

모든 토큰을 삭제하고 처음부터 다시 인증한다.

---

## 5. 설치 완료 후

설치와 인증이 모두 완료되면 `/nopal` 커맨드를 다시 실행한다.
커맨드가 자동으로 인증 상태를 감지하고 오케스트레이션 모드로 진입한다.

**확인 체크리스트:**
- [ ] `gws --version` → 버전 번호 출력
- [ ] `gws auth status` → 인증된 이메일 표시
- [ ] `/nopal` 실행 → 오케스트레이션 시작
