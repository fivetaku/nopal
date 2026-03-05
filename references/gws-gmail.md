# Gmail (v1)

```bash
gws gmail <resource> <method> [flags]
```

## 헬퍼 명령어

### +send -- 이메일 발송

```bash
gws gmail +send --to <EMAIL> --subject <SUBJECT> --body <TEXT>
```

| 플래그 | 필수 | 설명 |
|--------|------|------|
| `--to` | O | 수신자 이메일 |
| `--subject` | O | 제목 |
| `--body` | O | 본문 (plain text) |

```bash
gws gmail +send --to alice@example.com --subject 'Hello' --body 'Hi Alice!'
```

- RFC 2822 포맷 및 base64 인코딩 자동 처리
- HTML, 첨부파일, CC/BCC는 직접 API 사용: `gws gmail users messages send --json '...'`
- **write 명령** -- 실행 전 사용자 확인 필수

### +triage -- 받은편지함 요약

```bash
gws gmail +triage
```

| 플래그 | 기본값 | 설명 |
|--------|--------|------|
| `--max` | 20 | 최대 메시지 수 |
| `--query` | is:unread | Gmail 검색 쿼리 |
| `--labels` | - | 라벨 이름 포함 |

```bash
gws gmail +triage
gws gmail +triage --max 5 --query 'from:boss'
gws gmail +triage --format json | jq '.[].subject'
gws gmail +triage --labels
```

- 읽기 전용, 메일함 수정 없음
- 기본 출력은 table 형식

### +watch -- 새 이메일 스트리밍

```bash
gws gmail +watch --project <GCP_PROJECT_ID>
```

| 플래그 | 기본값 | 설명 |
|--------|--------|------|
| `--project` | - | GCP 프로젝트 ID (Pub/Sub용) |
| `--subscription` | - | 기존 Pub/Sub 구독 이름 |
| `--topic` | - | 기존 Pub/Sub 토픽 |
| `--label-ids` | - | 필터할 Gmail 라벨 ID (쉼표 구분) |
| `--max-messages` | 10 | pull 배치당 최대 메시지 |
| `--poll-interval` | 5 | pull 간격 (초) |
| `--msg-format` | full | 메시지 형식: full, metadata, minimal, raw |
| `--once` | - | 한 번 pull 후 종료 |
| `--cleanup` | - | 종료 시 Pub/Sub 리소스 삭제 |
| `--output-dir` | - | 메시지를 개별 JSON 파일로 저장 |

```bash
gws gmail +watch --project my-gcp-project
gws gmail +watch --project my-project --label-ids INBOX --once
gws gmail +watch --project my-project --cleanup --output-dir ./emails
```

- watch는 7일 후 만료 -- 재실행으로 갱신
- `--cleanup` 없으면 Pub/Sub 리소스 유지 (재연결용)

## API 리소스

### users

| 메서드 | 설명 |
|--------|------|
| `getProfile` | 현재 사용자 Gmail 프로필 |
| `stop` | push 알림 중지 |
| `watch` | push 알림 설정 |

### users.messages

```bash
# 메시지 목록 (검색)
gws gmail users messages list --params '{"userId": "me", "q": "from:boss is:unread"}'

# 메시지 상세
gws gmail users messages get --params '{"userId": "me", "id": "MSG_ID"}'

# 메시지 발송 (raw API)
gws gmail users messages send --params '{"userId": "me"}' --json '{"raw": "BASE64_RFC2822"}'

# 라벨 수정 (읽음/보관 처리)
gws gmail users messages modify --params '{"userId": "me", "id": "MSG_ID"}' --json '{"removeLabelIds": ["UNREAD"]}'
gws gmail users messages modify --params '{"userId": "me", "id": "MSG_ID"}' --json '{"removeLabelIds": ["INBOX"]}'

# 메시지 삭제
gws gmail users messages delete --params '{"userId": "me", "id": "MSG_ID"}'
```

### users.labels

```bash
# 라벨 목록
gws gmail users labels list --params '{"userId": "me"}' --format table

# 라벨 생성
gws gmail users labels create --params '{"userId": "me"}' --json '{"name": "Project-X"}'
```

### users.drafts

```bash
# 임시저장 목록
gws gmail users drafts list --params '{"userId": "me"}'

# 임시저장 생성
gws gmail users drafts create --params '{"userId": "me"}' --json '{"message": {"raw": "BASE64"}}'
```

### users.threads

```bash
# 스레드 목록
gws gmail users threads list --params '{"userId": "me", "q": "subject:meeting"}'

# 스레드 상세
gws gmail users threads get --params '{"userId": "me", "id": "THREAD_ID"}'
```

### users.settings

```bash
# 필터 생성
gws gmail users settings filters create --params '{"userId": "me"}' --json '{"criteria": {"from": "noreply@service.com"}, "action": {"addLabelIds": ["LABEL_ID"], "removeLabelIds": ["INBOX"]}}'

# 부재중 응답 설정
gws gmail users settings updateVacation --params '{"userId": "me"}' --json '{"enableAutoReply": true, "responseSubject": "Out of Office", "responseBodyPlainText": "..."}'
```

### users.messages.attachments

```bash
# 첨부파일 가져오기
gws gmail users messages attachments get --params '{"userId": "me", "messageId": "MSG_ID", "id": "ATTACHMENT_ID"}'
```

## 검색 쿼리 문법 (q 파라미터)

| 연산자 | 예시 |
|--------|------|
| `from:` | `from:alice@example.com` |
| `to:` | `to:team@company.com` |
| `subject:` | `subject:meeting` |
| `is:` | `is:unread`, `is:starred`, `is:important` |
| `label:` | `label:project-x` |
| `has:` | `has:attachment` |
| `after:` / `before:` | `after:2024/01/01 before:2024/12/31` |
| `newer_than:` / `older_than:` | `newer_than:7d` |
| 조합 | `from:boss is:unread has:attachment` |
