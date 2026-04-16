English | [한국어](README.ko.md)

# nopal

> **Google Workspace orchestration, powered by natural language.**

Turn a plain sentence into a coordinated action across Gmail, Calendar, Drive, Docs, Sheets, Slides, Meet, Tasks, and Chat — all without leaving Claude Code.

[Quick Start](#quick-start) • [Why nopal?](#why-nopal) • [How it works](#how-it-works) • [Services](#services) • [Requirements](#requirements)

---

## Quick Start

### 1. Add the marketplace (once)

```
/plugin marketplace add https://github.com/fivetaku/gptaku_plugins.git
```

### 2. Install nopal

```
/plugin install nopal
```

Restart Claude Code after installation.

### 3. Set up gws CLI (once)

nopal uses [gws CLI](https://github.com/googleworkspace/cli) to talk to Google Workspace. Install it first:

```bash
npm install -g @googleworkspace/cli
```

Then run the one-time OAuth setup in your terminal:

```bash
gws auth setup
```

This walks you through creating a GCP project, enabling the 9 Workspace APIs, and authorizing your Google account. After setup, log in:

```bash
gws auth login
```

After login, export credentials so Claude Code can use gws in headless mode:

```bash
gws auth export --unmasked 2>/dev/null | grep -v '^Using keyring' > ~/.config/gws/credentials.json
```

### 4. Run

```
/nopal
```

No arguments needed — nopal asks what you want. Or go directly:

```
/nopal schedule a team standup for tomorrow at 10am and email the attendees the agenda
/nopal check my unread emails and summarize the important ones
/nopal create a meeting notes doc and share it with last week's attendees
/nopal pull the Q1 sales data from Sheets and send a summary to the team chat
```

---

## Why nopal?

- **One command, any service** — describe what you want in plain language; nopal figures out which services to invoke and in what order
- **Dynamic composition** — not a fixed workflow library; services are selected and chained based on each request
- **Interview-driven** — if information is missing, nopal asks before acting (not after)
- **Read vs. write distinction** — read-only queries execute immediately; write and modify actions always get your confirmation first
- **Lives in Claude Code** — no new app, no browser tab, no context switch
- **No credentials in Claude** — gws CLI owns the OAuth tokens; nopal never touches them directly

---

## How it works

```
You: "schedule a team meeting tomorrow at 2pm and email attendees"
     │
     ▼
/nopal
     │
     ├─ gws not installed? → auto-install attempt / setup guidance
     │
     └─ gws ready → orchestration begins
          │
          ├─ 1. Parse intent      — which services are needed?
          ├─ 2. Interview         — fetch live data, ask only what's missing
          ├─ 3. Plan              — confirm write actions, skip read-only ones
          ├─ 4. Execute           — run gws commands sequentially
          └─ 5. Report            — summarize results + suggest next steps
```

Multi-service requests resolve naturally:

- "add attendees to tomorrow's meeting and send them the doc" → Calendar + Drive + Gmail
- "create a newsletter from the Sheets data and send it" → Sheets + Gmail
- "write meeting notes and post them to the team Chat space" → Docs + Chat

---

## Services

| Service | What nopal can do | Helper commands |
|---------|-------------------|-----------------|
| Gmail | Send, read, triage, watch | `+send`, `+triage`, `+watch` |
| Calendar | Create events, check agenda | `+insert`, `+agenda` |
| Drive | Upload files, manage sharing | `+upload` |
| Sheets | Read/append spreadsheet data | `+read`, `+append` |
| Docs | Read and write documents | `+write` |
| Slides | Create and edit presentations | — |
| Chat | Send messages to spaces | `+send` |
| Tasks | Manage to-do lists | — |
| Meet | Create meeting links, get participants and transcripts | — |

---

## Known Issues

| Issue | Status | Workaround |
|-------|--------|------------|
| Gmail trash 411 error | Fixed in gws 0.6.1+ | Use latest version |
| `+send` Korean encoding | gws CLI bug | Raw API encoding applied automatically |
| `gws auth export` log pollution | `Using keyring backend` mixed into JSON | `2>/dev/null \| grep -v '^Using keyring'` filter applied |

---

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [gws CLI](https://github.com/googleworkspace/cli) — `npm install -g @googleworkspace/cli`
- Google Workspace account + OAuth setup (`gws auth setup` + `gws auth login`)
- Node.js 18+

> When you run `/nopal` for the first time, it checks for gws and guides you through setup automatically.

---

## License

MIT

---

<div align="center">

**No Opal needed.**

</div>
