# Nopal

> **No + Opal = Nopal** — Google Workspace 9개 서비스를 자연어로 자동 조합

Google Opal이 Google 생태계 안에서만 동작하는 것과 달리, Nopal은 Claude Code 터미널 안에서 자연어로 9개 서비스를 동적으로 조합합니다.

---

## 이런 분을 위한 도구입니다

- 터미널을 떠나지 않고 Google Workspace를 조작하고 싶은 개발자
- 여러 서비스를 엮는 반복 작업(일정 확인 → 문서 작성 → 메일 발송)을 자동화하고 싶은 분
- Claude Code 안에서 Gmail, Calendar, Drive, Sheets 등을 바로 쓰고 싶은 분

---

## 어떻게 작동하나요?

```
사용자: "회의 준비해줘"
     │
     ▼
/nopal (환경 확인)
     │
     ├─ gws 미설치? → 자동 설치 시도 / 인증 안내
     │
     └─ gws 설치됨 → 오케스트레이션 시작
          │
          ├─ 1. 의도 파악: 어떤 서비스가 필요한지 분석
          ├─ 2. 인터뷰: 부족한 정보를 질문으로 수집
          ├─ 3. 계획 수립: 쓰기 작업만 확인, 읽기는 바로 실행
          ├─ 4. 실행: gws CLI로 서비스 순차 실행
          └─ 5. 결과 요약: 실행 결과 + 다음 액션 제안
```

---

## 설치 방법

### 1. 마켓플레이스 등록 (처음 한 번만)

```
/plugin marketplace add https://github.com/fivetaku/gptaku_plugins.git
```

### 2. 플러그인 설치

```
/plugin install nopal
```

> 설치 후에는 Claude Code를 **재시작**하세요.

### 처음 시작하기

```
/nopal
```

처음 실행하면 `gws` CLI 설치와 Google OAuth 인증을 안내합니다.

---

## 핵심 기능

### 1. 자연어 오케스트레이션

9개 서비스를 자연어 한마디로 자동 조합합니다. 미리 정해진 워크플로우가 아닌 동적 조합입니다.

```
/nopal 오늘 일정 알려줘
/nopal 내일 오후 2시에 팀 회의 잡고 참석자에게 메일 보내줘
/nopal 시트에 있는 수신자한테 뉴스레터 보내줘
/nopal 회의록을 Google Docs로 만들고 참석자에게 공유해줘
/nopal 오늘 마감인 할일 확인하고 못 끝낸 거 내일로 옮겨줘
```

### 2. 인터뷰 기반 정보 수집

모호한 요청도 괜찮습니다. 부족한 정보는 질문으로 채워서 완전한 워크플로우를 실행합니다.

### 3. 스마트 실행 분류

읽기 전용 조회는 확인 없이 바로 실행하고, 쓰기/변경 작업만 사용자 확인을 받습니다.

---

## 지원 서비스

| 서비스 | 주요 용도 | 단축 명령 |
|--------|----------|-----------|
| Gmail | 이메일 보내기/읽기/관리 | `+send`, `+triage`, `+watch` |
| Calendar | 일정/이벤트 관리 | `+insert`, `+agenda` |
| Drive | 파일/폴더/공유 관리 | `+upload` |
| Sheets | 스프레드시트 읽기/쓰기 | `+read`, `+append` |
| Docs | 문서 읽기/쓰기 | `+write` |
| Slides | 프레젠테이션 생성/편집 | — |
| Chat | 채팅 스페이스/메시지 | `+send` |
| Tasks | 할일 목록/태스크 관리 | — |
| Meet | 회의 링크 생성/참가자/녹화/스크립트 | — |

---

## 구성요소

| 구성요소 | 설명 |
|----------|------|
| 커맨드 | `/nopal` — 단일 진입점, 환경 확인 + 오케스트레이션 라우터 |
| 스킬 | `nopal-orchestrate` — 핵심 오케스트레이션 엔진 (5단계 워크플로우) |
| 스킬 | `nopal-setup` — gws CLI 설치 및 OAuth 인증 가이드 |
| 레퍼런스 | `references/gws-*.md` — 9개 서비스별 API 가이드 |

---

## 알려진 이슈

| 이슈 | 상태 | 우회 방법 |
|------|------|-----------|
| Gmail trash 411 에러 | gws 0.6.1+에서 수정됨 | 최신 버전 사용 |
| Gmail `+send` 한글 깨짐 | gws CLI 버그 | raw API 인코딩 (자동 적용) |
| `gws auth export` 로그 혼입 | `Using keyring backend` 로그가 JSON에 섞임 | `2>/dev/null \| grep -v '^Using keyring'` 필터 적용 |

---

## 요구사항

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 설치
- [gws CLI](https://github.com/googleworkspace/cli) 설치 (`npm install -g @googleworkspace/cli`)
- Google Workspace 계정 + OAuth 인증 (`gws auth login`)

> gws CLI 설치와 인증은 `/nopal` 첫 실행 시 자동으로 안내됩니다.

---

## 라이선스

MIT
