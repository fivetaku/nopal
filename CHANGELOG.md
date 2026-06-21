# Changelog

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
