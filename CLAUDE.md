# CLAUDE.md — index only (keep this file ≤150 lines; long content lives in docs/)

## Project
Static two-page web app, no build step, no package.json, no tests:
- `vote.html` — guests vote boy/girl. Firebase Realtime Database (config
  inline in the HTML); `localStorage` blocks duplicate votes.
- `results.html` — live tally and reveal animation, same Firebase backend.

Verification for this project = open the page in a browser and check the
console, not just reading the diff. Chromium is pre-installed in remote
sessions (Playwright `executablePath: '/opt/pw-browsers/chromium'`).

## Agent operating system (load on demand — do NOT read all of these up front)
Protocols written 2026-07-04 by a Fable 5 session for cheaper models.
Open only the file whose trigger matches:

| File | Read it when |
| --- | --- |
| `docs/agent-os/00-diagnosis.md` | Starting any multi-step task (top 3 failure modes + fixes) |
| `docs/agent-os/10-model-dispatch.md` | Before spawning any subagent, or choosing model/effort |
| `docs/agent-os/20-judgment-rubrics.md` | Unsure whether to escalate, stop, ask the user, or call it done |
| `docs/agent-os/30-prompt-templates.md` | Writing a subagent task prompt |
| `docs/agent-os/40-maintenance.md` | Before editing this file or anything in `docs/agent-os/` |
| `docs/agent-os/LESSONS.md` | After something went wrong (append), or when a task feels familiar (check) |
| `docs/agent-os/90-letter.md` | First session in a new context window, or when a protocol seems wrong |

Predefined subagents in `.claude/agents/`: `scout` (Haiku, search),
`worker` (Sonnet, implementation), `verifier` (Sonnet, fresh-context checks).
Note: built-in `Explore` and `Plan` subagents do NOT load this CLAUDE.md —
put needed context in their prompt.

## Hard rules (always apply, no file lookup needed)
1. Don't paste more than ~40 lines of file/log/tool output into the
   conversation. Summarize and cite `path:line`. Fan-out searches go to a
   subagent, not the main thread. (Caps by situation: ~40 lines for mid-task
   pastes; ≤10 quoted lines in a user-facing report; subagent payloads over
   ~30 lines go to a file, return the path.)
2. Before claiming anything is done or working: run the verification you
   defined at task start. No evidence, no claim — write "NOT VERIFIED" if
   you couldn't run it.
3. Never write model IDs, API params, or config keys from memory. Sources:
   verified-facts block in `docs/agent-os/10-model-dispatch.md`, the
   `claude-api` skill, or official docs.
4. In remote (ephemeral) sessions, commit and push work before the session
   ends — anything unpushed is lost.
5. After any failure worth remembering, append one entry to
   `docs/agent-os/LESSONS.md` (format defined in `40-maintenance.md`).
6. SPOILER GUARD: this is a gender reveal — never print vote tallies,
   database contents, or anything implying the result into chat, commits,
   or logs unless the user explicitly asks for the numbers. Verification
   evidence must be structural (element rendered, console clean), never
   the actual counts.
