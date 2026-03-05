# Nopal

> **No + Opal = Nopal** — Google Opal? 이제 필요 없다.

Google Workspace CLI(`gws`)가 Claude Code 안으로 들어왔습니다.

Gmail, Calendar, Drive, Sheets, Docs, Slides, Chat, Tasks, Meet — 9개 서비스를 자연어 한마디로 자동 조합해서 실행합니다. 서비스 하나씩 따로 열 필요 없이, "회의 준비해줘"라고 말하면 캘린더 확인 → 문서 생성 → 메일 발송까지 한번에.

## 뭐가 다른가

Google은 [Opal](https://opal.google/)을 만들어서 Workspace 서비스 간 자동화를 시도했습니다. 하지만 Opal은 Google 생태계 안에서만 동작하고, 개발자의 로컬 워크플로우와는 완전히 분리되어 있습니다.

Nopal은 다릅니다:

- **Claude Code 안에서 동작합니다.** 터미널을 떠나지 않고 Google Workspace를 조작합니다.
- **자연어로 오케스트레이션합니다.** "시트 데이터로 메일 보내줘" 한마디면 Sheets 읽기 → Gmail 발송을 자동 조합합니다.
- **동적 조합입니다.** 미리 정해진 워크플로우만 실행하는 게 아닙니다. 어떤 요청이든 9개 서비스를 조합해서 처리합니다.
- **인터뷰 기반입니다.** 모호한 요청도 괜찮습니다. 부족한 정보는 질문으로 채워서 완전한 워크플로우를 실행합니다.

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

## 사용법

### 설치

```
/plugin install nopal
```

### 시작

```
/nopal
```

처음 실행하면 `gws` CLI 설치와 Google 인증을 안내합니다.

### 예시

```
/nopal 오늘 일정 알려줘

/nopal 내일 오후 2시에 팀 회의 잡고 참석자에게 메일 보내줘

/nopal 시트에 있는 수신자한테 뉴스레터 보내줘

/nopal 회의록을 Google Docs로 만들고 참석자에게 공유해줘

/nopal 오늘 마감인 할일 확인하고 못 끝낸 거 내일로 옮겨줘
```

단일 서비스부터 여러 서비스를 엮는 복잡한 요청까지, 어떤 조합이든 동적으로 처리합니다.

## 동작 원리

```
사용자: "회의 준비해줘"
     │
     ▼
/nopal (환경 확인)
     │
     ├─ gws 미설치? → 설치/인증 안내
     │
     └─ gws 설치됨 → 오케스트레이션 시작
          │
          ├─ 1. 의도 파악: 어떤 서비스가 필요한지 분석
          ├─ 2. 인터뷰: 부족한 정보를 질문으로 수집
          ├─ 3. 계획 수립: 실행 계획을 보여주고 확인
          ├─ 4. 실행: gws CLI로 서비스 순차 실행
          └─ 5. 결과 요약: 실행 결과 + 다음 액션 제안
```

## 전제 조건

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 설치
- [gws CLI](https://github.com/googleworkspace/cli) 설치 (`npm install -g @googleworkspace/cli`)
- Google Workspace 계정 + OAuth 인증 (`gws auth login`)

> gws CLI 설치와 인증은 `/nopal` 첫 실행 시 자동으로 안내됩니다.

## License

MIT
