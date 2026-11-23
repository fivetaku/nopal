# Changelog

## 0.8.0 — 2026-11-23
- Codex에서도 같은 스킬로 동작: `/nopal` 명령에만 있던 환경 확인(gws 설치·인증)과 요청 파싱 절차를 `nopal-orchestrate` 스킬로 옮기고 명령은 얇게 만들었다. Claude Code 동작은 그대로다.
- Codex 질문 도구 사용법과 경로·인터랙티브 인증 차이를 `references/codex.md`에 추가.

## 0.7.8 — 2026-11-22
- Korean-subject mail goes through a raw API helper.
- Outgoing steps stop if a dependency failed.

## 0.7.6 — 2026-10-13
- **Routed through gptaku-core.** Declares `gptaku-core` as a dependency and adds `routing.json`, so requests that match this plugin's skills are pointed to them before Claude handles the work itself or reaches for native tools. Existing users: run `/reload-plugins` after updating, or reinstall the plugin, so gptaku-core gets installed. Opt out with `GPTAKU_ROUTER=off`.

## 0.7.5 — 2026-08-24

- **Fix: first-run setup could wipe `settings.json`** — if the file was corrupted or contained comments (JSONC), the shared update-notifier installer re-wrote it as an empty object plus the hook, silently destroying all user settings. It now refuses to write when parsing fails and writes atomically (tmp + rename). Marketplace-wide propagation of the fix found in the ddiring v0.1.1 external review; reproduction-verified.

## 0.7.2 — 2026-06-21

- The GitHub-star prompt is shown in the user's current language; on a fresh session with no language signal yet, it falls back to the language detected from your recent Claude sessions (else English).
- GitHub star is now **opt-in** — on first run the command asks once via AskUserQuestion (`네, ⭐ 눌러주기` / `아니요`) instead of auto-starring. The star logic moved into `setup.sh` and records the choice (`~/.gptaku-setup/<plugin>.star.json`) so it never re-asks. `setup.sh` no longer stars anything automatically.
- Docs: corrected the README 'no credentials in Claude' line to match the actual plaintext credential export, and added a `chmod 600` step for the exported token file.

## [0.6.3] - 2026-05-04

### Changed
- `nopal-orchestrate/SKILL.md`:577 — "다음 액션 제안 2-3개" hard floor → "도메인에 의미 있는 후속이 있을 때만 (강제 개수 없음, 0개 OK)" (fossil v3 처치)

### Preserved
- 9 GWS 서비스 capability 테이블 (Gmail/Calendar/Drive/Docs/Sheets/Forms/Slides/Photos/Tasks)
- gws CLI command schema (자동화 contract)
- lazy-load reference 패턴
- 결과 파싱 → 다음 단계 ID/값 추출 결정성

## [0.6.2] (이전 버전)

본 CHANGELOG는 v0.6.3에서 신설. 이전 버전 변경 이력은 git log 참조.
