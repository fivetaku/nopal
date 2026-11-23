# Codex에서 실행할 때 (nopal)

`skills/nopal-orchestrate/SKILL.md` 절차는 그대로 따르고, 아래 차이만 적용한다.

## 선택지 질문 = 질문 도구 (이번 세션 도구 목록으로 판단)

1. `request_user_input_async`가 있으면 선택지 질문 1콜 = 이 도구 1콜. 문항 ≤4개를 한 번에 넣는다.
   - 형식: `{"questions":[{"title":"헤더 — 한 줄 질문","options":["추천안 (추천) — 설명","대안 — 설명"]}]}`
   - 문항에는 `title`·`options`(문자열 배열)만. `question`·`header`·`id`를 넣으면 파싱 오류가 난다.
   - 추천안을 맨 앞에. "기타" 옵션은 넣지 않는다(자유 입력 칸이 자동으로 붙는다).
   - 부른 뒤 "위 질문에 답해 주시면 이어서 진행합니다" 한 줄만 쓰고 턴을 끝낸다. 답이 오기 전에 쓰기·발송 작업을 진행하지 않는다.
2. Plan 모드라 `request_user_input`만 있으면: 1콜 3문항까지. `header`(12자 이내)·`id`(snake_case)·`question`, 선택지 2~3개(`label`·`description`), 추천 라벨 끝에 `(Recommended)`.
3. 둘 다 없으면 채팅에 번호 블록(추천안 1번, 마지막은 "문장으로 직접 적기")을 내고 "Q1-1, Q2-2"처럼 답을 받는다. 모르면 1번으로 진행한다고 덧붙인다.
4. `codex exec`처럼 사람이 답할 수 없는 실행이면 묻지 않는다. 조회는 진행하되, 메일 발송·일정 생성·삭제 같은 쓰기·변경은 하지 않고 계획만 보고한다.

## 차이

- 정책 정본: `shared/questioning-policy.md` §7. 선택지 앞에 보여 줄 미리보기(`preview`)는 도구 호출 직전 채팅에 짧게 쓴다.
- `${CLAUDE_PLUGIN_ROOT}`는 이 플러그인 루트로 바꿔 읽는다. `gws auth setup`·`gws auth login`은 인터랙티브라 Codex도 실행하지 못한다 — 사용자 터미널에서 하도록 안내하고 중단한다.
- 첫 실행 설정(`setup/setup.sh`)은 Claude Code 전용이다. Codex에서는 실행하지 않는다.
- `/nopal` 명령 대신 스킬이 직접 트리거된다(스킬 `nopal-orchestrate`). 다른 스킬을 부르는 단계는 그 스킬의 SKILL.md를 열어 따른다.
