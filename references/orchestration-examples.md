# 오케스트레이션 복합 조합 예시

코어 9개 서비스를 조합하는 실전 ExecutionPlan 예시.
SKILL.md의 5단계 워크플로우(의도 파악 → 인터뷰 → 계획 → 실행 → 요약)를 따른다.

---

## 예시 1: 회의 준비 자동화

요청: "내일 팀 회의 준비해줘"

```
ExecutionPlan:
| # | 서비스 | 작업 | 의존 |
|---|--------|------|------|
| 1 | Calendar | 내일 팀 회의 일정 조회 | - |
| 2 | Docs | 회의 안건 문서 생성 | 1 |
| 3 | Drive | 문서를 참석자에게 공유 | 1, 2 |
| 4 | Gmail | 참석자에게 안건 메일 발송 | 1, 2, 3 |
```

```bash
# Step 1: 내일 일정 조회
gws calendar +agenda --days 2 --format json

# Step 2: 회의록 문서 생성
DOC_ID=$(gws docs documents create --json '{"title": "팀 회의 안건 - 2026-03-06"}' --format json | jq -r '.documentId')

# Step 3: 참석자에게 문서 공유
gws drive permissions create --params "{\"fileId\": \"$DOC_ID\"}" --json '{
  "role": "writer",
  "type": "user",
  "emailAddress": "alice@example.com"
}' --format json

# Step 4: 메일 발송
gws gmail +send --to "alice@example.com" --subject "내일 팀 회의 안건" --body "안건 문서를 공유드립니다: https://docs.google.com/document/d/$DOC_ID"
```

---

## 예시 2: 주간 보고서 자동화

요청: "이번 주 매출 시트에서 합계 구하고 보고서 만들어서 팀장에게 보내줘"

```
ExecutionPlan:
| # | 서비스 | 작업 | 의존 |
|---|--------|------|------|
| 1 | Sheets | 매출 시트 데이터 읽기 | - |
| 2 | Docs | 매출 요약 보고서 문서 생성 | 1 |
| 3 | Gmail | 팀장에게 보고서 메일 발송 | 2 |
```

```bash
# Step 1: 시트 데이터 읽기
SALES_DATA=$(gws sheets +read --spreadsheet "SHEET_ID" --range "A1:D50" --format json)

# Step 2: 보고서 문서 생성 후 내용 추가
DOC_ID=$(gws docs documents create --json '{"title":"주간 매출 보고서 - 2026 W10"}' --format json | jq -r '.documentId')
gws docs +write --document "$DOC_ID" --text "이번 주 매출 요약..."

# Step 3: 팀장에게 메일 발송
gws gmail +send --to "manager@example.com" --subject "주간 매출 보고서" --body "보고서 링크: https://docs.google.com/document/d/$DOC_ID"
```

---

## 예시 3: 할일 관리 + 알림

요청: "오늘 마감인 할일 확인하고 못 끝낸 거 내일로 옮겨줘"

```
ExecutionPlan:
| # | 서비스 | 작업 | 의존 |
|---|--------|------|------|
| 1 | Tasks | 오늘 마감 태스크 조회 | - |
| 2 | Tasks | 미완료 태스크 마감일을 내일로 수정 | 1 |
| 3 | Chat | 미완료 태스크 알림 메시지 전송 (선택) | 1 |
```

```bash
# Step 1: 태스크 조회
gws tasks tasks list --params '{"tasklist": "TASKLIST_ID"}' --format json

# Step 2: 미완료 태스크 마감일 수정
gws tasks tasks patch --params '{"tasklist": "TASKLIST_ID", "task": "TASK_ID"}' --json '{
  "due": "2026-03-06T00:00:00Z"
}' --format json
```

---

## 결과 리포트 형식 예시

```
## 실행 결과

### 완료된 작업
1. Calendar: 내일 오후 2시에 "팀 회의" 일정 생성 완료
   - 참석자: alice@example.com, bob@example.com
   - Google Meet 링크: https://meet.google.com/xxx-xxxx-xxx

2. Docs: "팀 회의 안건" 문서 생성 완료
   - 문서 링크: https://docs.google.com/document/d/xxx

3. Gmail: 참석자 2명에게 회의 안건 메일 발송 완료
   - 제목: "내일 팀 회의 안건"

### 실패한 작업
(없음)

### 다음에 할 수 있는 작업
- "회의 끝나면 회의록 정리해줘"
- "참석자에게 후속 할일 배정해줘"
```
