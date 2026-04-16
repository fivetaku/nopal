[English](README.md) | 한국어

# nopal

> **자연어로 작동하는 Google Workspace 오케스트레이터**

한 문장으로 Gmail, Calendar, Drive, Docs, Sheets, Slides, Meet, Tasks, Chat을 조합해 실행합니다. Claude Code 터미널에서 벗어날 필요 없이.

[빠른 시작](#빠른-시작) • [왜 nopal인가?](#왜-nopal인가) • [작동 원리](#작동-원리) • [지원 서비스](#지원-서비스) • [요구사항](#요구사항)

---

## 빠른 시작

### 1. 마켓플레이스 등록 (처음 한 번만)

```
/plugin marketplace add https://github.com/fivetaku/gptaku_plugins.git
```

### 2. nopal 설치

```
/plugin install nopal
```

설치 후 Claude Code를 재시작하세요.

### 3. gws CLI 설치 (처음 한 번만)

nopal은 [gws CLI](https://github.com/googleworkspace/cli)를 통해 Google Workspace와 통신합니다. 먼저 설치하세요:

```bash
npm install -g @googleworkspace/cli
```

그런 다음 터미널에서 일회성 OAuth 설정을 진행합니다:

```bash
gws auth setup
```

GCP 프로젝트 생성, 9개 Workspace API 활성화, Google 계정 인증 순서로 진행됩니다. 설정 완료 후 로그인:

```bash
gws auth login
```

로그인 후 Claude Code가 headless 환경에서 gws를 사용할 수 있도록 credentials를 내보냅니다:

```bash
gws auth export --unmasked 2>/dev/null | grep -v '^Using keyring' > ~/.config/gws/credentials.json
```

### 4. 실행

```
/nopal
```

인자 없이 실행하면 nopal이 먼저 물어봅니다. 바로 요청해도 됩니다:

```
/nopal 내일 오후 2시에 팀 회의 잡고 참석자에게 안건 메일 보내줘
/nopal 읽지 않은 이메일 중 중요한 것만 요약해줘
/nopal 회의록 문서 만들고 지난주 참석자들에게 공유해줘
/nopal 1분기 매출 데이터 시트에서 합계 구해서 팀 채팅으로 보내줘
```

---

## 왜 nopal인가?

- **명령 하나로 어떤 서비스든** — 자연어로 요청하면 어떤 서비스가 필요한지, 어떤 순서로 실행할지 스스로 판단합니다
- **동적 조합** — 미리 정해진 워크플로우가 아닌, 요청마다 서비스를 선택하고 체이닝합니다
- **인터뷰 기반** — 정보가 부족하면 실행 전에 먼저 물어봅니다. 실행 후가 아닙니다
- **읽기/쓰기 구분** — 조회는 확인 없이 바로 실행, 쓰기와 변경 작업은 항상 확인을 받습니다
- **Claude Code 안에서** — 새 앱도, 브라우저 탭도, 컨텍스트 전환도 없습니다
- **Claude에 credentials 없음** — gws CLI가 OAuth 토큰을 직접 관리합니다. nopal은 토큰을 다루지 않습니다

---

## 작동 원리

```
사용자: "내일 오후 2시에 팀 회의 잡고 참석자에게 메일 보내줘"
     │
     ▼
/nopal
     │
     ├─ gws 미설치? → 자동 설치 시도 / 설치 가이드 안내
     │
     └─ gws 준비됨 → 오케스트레이션 시작
          │
          ├─ 1. 의도 파악    — 어떤 서비스가 필요한지 분석
          ├─ 2. 인터뷰       — 실시간 데이터 조회, 부족한 정보만 질문
          ├─ 3. 계획 수립    — 쓰기 작업 확인, 읽기 전용은 바로 실행
          ├─ 4. 실행         — gws 명령어로 서비스 순차 실행
          └─ 5. 결과 요약    — 실행 결과 + 다음 액션 제안
```

복합 요청도 자연스럽게 처리됩니다:

- "내일 회의 참석자 추가하고 문서 보내줘" → Calendar + Drive + Gmail
- "시트 데이터로 뉴스레터 만들어서 발송해줘" → Sheets + Gmail
- "회의록 작성하고 팀 Chat 스페이스에 올려줘" → Docs + Chat

---

## 지원 서비스

| 서비스 | nopal이 할 수 있는 것 | 단축 명령 |
|--------|----------------------|-----------|
| Gmail | 보내기/읽기/트리아지/감시 | `+send`, `+triage`, `+watch` |
| Calendar | 이벤트 생성/일정 확인 | `+insert`, `+agenda` |
| Drive | 파일 업로드/공유 관리 | `+upload` |
| Sheets | 스프레드시트 읽기/쓰기 | `+read`, `+append` |
| Docs | 문서 읽기/쓰기 | `+write` |
| Slides | 프레젠테이션 생성/편집 | — |
| Chat | 스페이스 메시지 전송 | `+send` |
| Tasks | 할일 목록 관리 | — |
| Meet | 회의 링크 생성/참가자/녹화/스크립트 | — |

---

## 알려진 이슈

| 이슈 | 상태 | 우회 방법 |
|------|------|-----------|
| Gmail trash 411 에러 | gws 0.6.1+에서 수정됨 | 최신 버전 사용 |
| `+send` 한글 깨짐 | gws CLI 버그 | raw API 인코딩 자동 적용 |
| `gws auth export` 로그 혼입 | `Using keyring backend` 로그가 JSON에 섞임 | `2>/dev/null \| grep -v '^Using keyring'` 필터 적용 |

---

## 요구사항

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [gws CLI](https://github.com/googleworkspace/cli) — `npm install -g @googleworkspace/cli`
- Google Workspace 계정 + OAuth 인증 (`gws auth setup` + `gws auth login`)
- Node.js 18 이상

> `/nopal` 첫 실행 시 gws 설치 여부를 자동으로 확인하고 설치와 인증을 안내합니다.

---

## 라이선스

MIT

---

<div align="center">

**No + Opal = Nopal. Google Opal 없이도 충분합니다.**

</div>
