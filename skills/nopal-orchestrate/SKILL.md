---
name: nopal-orchestrate
description: Nopal 핵심 오케스트레이션 엔진. 사용자의 자연어 요청을 분석하여 Google Workspace 9개 서비스를 동적으로 조합 실행합니다. "/nopal", "메일 보내줘", "일정 확인", "회의 준비", "스프레드시트 만들어" 같은 요청에 사용됩니다.
---

# Nopal 오케스트레이션 엔진

> 이 문서는 커맨드(`commands/nopal.md`)에서 Read로 읽히는 **참고 자료**다.
> 사용자 요청을 받아 Google Workspace 서비스를 동적으로 조합하고 실행하는 핵심 로직이다.

---

## 지원 서비스 (9개)

| 서비스 | 헬퍼 명령어 | 주요 용도 |
|--------|------------|----------|
| Gmail | `+send`, `+triage`, `+watch` | 이메일 보내기/읽기/관리 |
| Calendar | `+insert`, `+agenda` | 일정/이벤트 관리 |
| Drive | `+upload` | 파일/폴더/공유 관리 |
| Sheets | `+read`, `+append` | 스프레드시트 읽기/쓰기 |
| Docs | `+write` | 문서 읽기/쓰기 |
| Slides | (직접 API) | 프레젠테이션 생성/편집 |
| Chat | `+send` | 채팅 스페이스/메시지 |
| Tasks | (직접 API) | 할일 목록/태스크 관리 |
| Meet | (직접 API) | 회의 링크 생성/참가자/녹화/스크립트 |

---

## 워크플로우 (5단계)

### Step 1: 의도 파악

사용자의 자연어 요청에서 키워드/의도를 추출하여 필요한 서비스를 식별한다.

| 키워드/의도 | 매핑 서비스 |
|------------|-----------|
| 메일, 이메일, 보내, 수신, 답장 | Gmail |
| 일정, 회의, 미팅, 약속, 캘린더, 스케줄 | Calendar |
| 파일, 폴더, 업로드, 다운로드, 공유, 드라이브 | Drive |
| 스프레드시트, 엑셀, 표, 데이터, 시트, 합계, 매출 | Sheets |
| 문서, 글, 작성, 편집, 보고서, 회의록 | Docs |
| 발표, 슬라이드, 프레젠테이션, PPT | Slides |
| 채팅, 메시지, 알림 | Chat |
| 할일, 태스크, 체크리스트, 투두 | Tasks |
| 화상회의, 미트, 회의 링크, 녹화, 참가자, 스크립트 | Meet |

하나의 요청에 여러 서비스가 관여할 수 있다.
예: "회의 참석자에게 메일 보내줘" → Calendar + Gmail

**실행 유형 판별:**

| 유형 | 설명 | 예시 |
|------|------|------|
| 단순 조회 | 정보만 읽기 | "오늘 일정 알려줘" |
| 단순 실행 | 한 서비스 동작 | "메일 보내줘" |
| 복합 조합 | 여러 서비스 체이닝 | "회의록 작성 후 참석자에게 공유" |

---

### Step 2: 인터뷰 (부족한 정보 수집)

**원칙: 질문은 최소화한다. 필요한 것만 묻는다.**

요청 실행에 필요한 파라미터를 나열하고, 이미 주어진 것과 부족한 것을 구분한다.

**기존 데이터 조회 후 옵션 제시:** 가능하면 Google Workspace에서 데이터를 먼저 조회하여 AskUserQuestion 옵션에 반영한다. 예: "어떤 회의를 준비할까요?" 대신 Calendar 조회 → 회의 목록을 옵션으로 제시.

**AskUserQuestion 규칙:**
- 텍스트로 질문하지 않는다. 반드시 AskUserQuestion 도구를 사용한다.
- 질문은 한 번에 묶어서 한다 (라운드트립 최소화).
- 기본값으로 해결 가능한 파라미터는 질문하지 않고 기본값 사용 (회의 1시간, 이메일 일반 텍스트, 공유 권한 "보기 가능").
- 모호한 요청의 경우에만 명확화 질문을 한다.

---

### Step 3: 계획 수립

**3-1. 레퍼런스 로딩** — 필요한 서비스의 reference 파일만 선택적으로 Read한다.

```
Read ${CLAUDE_PLUGIN_ROOT}/references/gws-shared.md          (항상 읽기)
Read ${CLAUDE_PLUGIN_ROOT}/references/gws-{서비스명}.md       (필요한 서비스만)
```

references/ 파일이 아직 없을 수 있다. 없으면 이 SKILL.md의 서비스 테이블과 헬퍼 목록을 기반으로 동작한다.

**3-2. 프리셋 패턴 확인** — `workflows.md`, `recipes.md`가 있으면 참조하여 검증된 패턴을 우선 사용. 없어도 동적으로 새 조합을 생성할 수 있다.

**3-3. ExecutionPlan 동적 생성** — 실행할 gws 명령어를 순서대로 나열:

| 필드 | 설명 |
|------|------|
| order | 실행 순서 (1, 2, 3, ...) |
| service | 사용하는 서비스 |
| command | 실제 gws 명령어 |
| depends_on | 이전 단계 결과 의존 시 해당 order 번호 |
| description | 이 단계가 하는 일 (한글) |

**3-4. 사용자 확인 (조건부)**

**읽기 전용 단순 조회는 확인 없이 바로 실행한다.**

| 실행 유형 | 확인 | 이유 |
|-----------|------|------|
| 단순 조회 (read-only) | **생략 → 바로 Step 4** | 부작용 없음 |
| 단순 실행 (write 1개) | 확인 필요 | 데이터 변경 |
| 복합 조합 (2+ 서비스) | 확인 필요 | 여러 서비스 변경 |

단순 조회 판별: 서비스 1개 + `list`/`get`/`+triage`/`+agenda` 등 읽기 전용 명령어만 사용.

단순 조회가 아닌 경우, ExecutionPlan을 마크다운 테이블로 보여주고 AskUserQuestion으로 확인받는다 ("실행" / "수정" / "취소").

---

### Step 4: 실행

**명령어 실행 규칙:**

| 규칙 | 설명 |
|------|------|
| 헬퍼 우선 | 헬퍼 명령어가 있는 서비스는 헬퍼를 우선 사용 |
| JSON 출력 | 모든 명령어에 `--format json` 플래그 사용 |
| 결과 파싱 | 각 단계 실행 후 JSON 결과에서 다음 단계에 필요한 ID/값 추출 |
| 에러 격리 | 한 단계가 실패해도 나머지 단계는 계속 진행 |

**서비스별 명령어 상세 →** 각 서비스의 reference 파일 참조:
```
${CLAUDE_PLUGIN_ROOT}/references/gws-gmail.md
${CLAUDE_PLUGIN_ROOT}/references/gws-calendar.md
${CLAUDE_PLUGIN_ROOT}/references/gws-drive.md
${CLAUDE_PLUGIN_ROOT}/references/gws-sheets.md
${CLAUDE_PLUGIN_ROOT}/references/gws-docs.md
${CLAUDE_PLUGIN_ROOT}/references/gws-slides.md
${CLAUDE_PLUGIN_ROOT}/references/gws-chat.md
${CLAUDE_PLUGIN_ROOT}/references/gws-tasks.md
${CLAUDE_PLUGIN_ROOT}/references/gws-meet.md
```

**결과 체이닝 패턴:** 이전 단계 출력에서 `jq`로 다음 단계에 필요한 ID를 추출한다.

```bash
# 예: 파일 생성 → ID 추출 → 공유
FILE_ID=$(gws docs documents create --json '{"title": "회의록"}' --format json | jq -r '.documentId')
gws drive permissions create --params "{\"fileId\": \"$FILE_ID\"}" --json '{"role":"writer","type":"user","emailAddress":"user@example.com"}' --format json
```

**에러 처리:** 실패한 단계를 기록하고, 의존하는 후속 단계만 스킵. 의존하지 않는 단계는 계속 진행. 전체를 중단하지 않는다.

---

### Step 5: 결과 요약

JSON 덤프를 하지 않는다. 사람이 읽기 쉬운 요약으로 정리:

| 규칙 | 설명 |
|------|------|
| 링크 포함 | 생성된 문서, 일정 등의 직접 링크 제공 |
| 성공/실패 구분 | 각 단계의 성공/실패를 명확히 표시 |
| 실패 원인 | 실패한 단계는 에러 원인 + 해결 방법 안내 |
| 다음 액션 제안 | 이어서 할 수 있는 관련 작업 2-3개 제안 |
| JSON 금지 | 원시 JSON을 그대로 출력하지 않는다 |

**복합 조합 예시 →** `${CLAUDE_PLUGIN_ROOT}/references/orchestration-examples.md` 참조

---

## 절대 하지 마 (DO NOT)

1. **gws CLI를 거치지 않고 Google API를 직접 HTTP 호출하지 마.** 모든 Google Workspace 상호작용은 반드시 `gws` 명령어를 통한다.
2. **OAuth 토큰이나 API 키를 하드코딩하지 마.** gws CLI가 토큰을 자체 관리한다.
3. **AskUserQuestion을 allowed-tools에 넣지 마.** auto-approve 버그로 UI가 렌더링되지 않는다.
4. **references/ 파일을 실행 시점에 전부 읽지 마.** 필요한 서비스의 reference만 선택적으로 Read한다.
5. **사용자에게 gws 명령어를 직접 타이핑하라고 안내하지 마.** Nopal이 Bash로 대신 실행한다.
6. **프리셋 패턴에 없다고 거부하지 마.** references를 참고해서 동적으로 새 조합을 생성한다.
7. **실행 중 에러가 나면 전체를 중단하지 마.** 실패한 단계만 보고하고 나머지는 계속 진행한다.
8. **gws CLI 미설치 시 안내만 하지 마.** `npm install -g @googleworkspace/cli`로 자동 설치한다. 실패 시에만 수동 안내.
9. **원시 JSON을 결과로 출력하지 마.** 사람이 읽기 쉬운 요약으로 정리한다.
10. **사용자 확인 없이 실행하지 마.** 반드시 계획을 보여주고 확인받은 후 실행한다.

---

## 항상 해 (ALWAYS DO)

1. **`/nopal` 실행 시 `gws auth status`로 인증 상태를 먼저 확인한다.**
2. **사용자 요청이 모호하면 반드시 AskUserQuestion으로 명확화한다.** 텍스트로 질문하지 않는다.
3. **실행 계획을 세운 뒤 사용자에게 미리보기를 보여주고 확인받은 후 실행한다.**
4. **각 gws 명령어 실행 후 결과를 파싱하여 다음 단계에 필요한 ID/값을 추출한다.**
5. **최종 결과를 사람이 읽기 쉬운 요약으로 정리한다.** JSON 덤프 금지.
6. **에러 발생 시 친절한 한글 메시지로 안내하고 해결 방법을 제시한다.**
7. **references/에서 해당 서비스의 정확한 gws 명령어 구문을 확인한 후 실행한다.**
8. **헬퍼 명령어가 있는 서비스는 헬퍼를 우선 사용한다.** 직접 API는 헬퍼로 불가능할 때만.
9. **`--format json`을 사용하여 구조화된 출력을 획득한다.** 파싱과 체이닝을 위해 필수.
10. **실행 결과에 생성된 리소스의 직접 링크를 포함한다.** (문서 URL, 일정 링크 등)
