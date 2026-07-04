# 00 — Harness Diagnosis: Top 3 Failure Modes and Exact Fixes

Written 2026-07-04 by a Fable 5 session for cheaper models (Sonnet/Opus/Haiku).
Read this once at the start of any multi-step task. Each fix below is a
procedure, not advice — follow it literally.

---

## Failure #1 — Context flooding: raw tool output burns the budget

**What happens:** You Read whole files, cat logs, or fetch web pages into the
main conversation. Every token you pull in is re-paid on EVERY subsequent
turn. A 2,000-line file read early in a session can cost more than all your
reasoning combined, and it also pushes the original task instructions toward
summarization, which triggers Failure #2.

**The fix (follow literally):**
1. Before any `Read`: if you know roughly what you need, use `Grep` first,
   then `Read` with `offset`/`limit` around the match. Never read more than
   ~100 lines when you need one function.
2. If you must inspect more than 3 files, or you don't know where the answer
   lives: do NOT search in the main conversation. Spawn an `Explore` subagent
   (or the project `scout` agent) and ask for conclusions + `file:line`
   citations only. See `docs/agent-os/10-model-dispatch.md`.
3. Logs and command output: never paste raw. Pipe through `grep`/`tail`
   (`... 2>&1 | grep -iE 'error|fail' | head -30`). If a tool result was
   persisted to a file because it was too large, grep that file — do not
   re-fetch.
4. When reporting to the user, cite `path:line`; quote at most ~10 lines.

**Self-check:** if any single tool result in your conversation is longer than
one screen and you used less than a quarter of it, you triggered this failure.

---

## Failure #2 — Goal drift: losing the acceptance criteria mid-task

**What happens:** After many tool calls, an error detour, or a context
summarization, you start optimizing for "finish the thing in front of me"
instead of what was asked. Classic endings: you fix a side issue and declare
victory; you end your turn with a plan instead of doing the work; you answer
a related-but-different question.

**The fix (follow literally):**
1. At task start, before the first tool call, write acceptance criteria down
   where they survive: use `TaskCreate` (preferred), or write 3–5 bullets to
   a scratch file (use the session scratchpad directory). Format:
   - DONE MEANS: <observable outcome, e.g. "vote.html shows X and console has no errors">
   - MUST NOT: <what must not change/break>
   - VERIFY BY: <exact command or action>
2. Before ending ANY turn, re-read those bullets and check each one. A bullet
   you cannot check off with evidence means the turn is not over — keep working.
3. If you notice the conversation was summarized (earlier detail feels fuzzy),
   re-read the criteria file/task BEFORE the next action, not after.
4. If mid-task you discover the real problem is different from the stated
   task, don't silently pivot: state the finding in one paragraph, then apply
   the "stop and ask" rubric in `docs/agent-os/20-judgment-rubrics.md`.

**Self-check:** can you quote your acceptance criteria right now without
scrolling? If not, re-read them before the next tool call.

---

## Failure #3 — Unverified assertions: "done" without evidence, facts from memory

**What happens:** You claim a fix works without running anything; you fill in
model IDs, API parameters, or config keys from training memory (often stale);
you trust that your own edit did what you intended. Cheaper models do this
more, and it is the single biggest source of silently-wrong output.

**The fix (follow literally):**
1. No evidence, no claim. Every "done/works/fixed" in your report must be
   preceded in the same session by the command you ran and what it printed.
   If you can't run it, write "NOT VERIFIED: <what you'd run>" instead of
   claiming success.
2. Facts lookup table — before writing any of these, check the source:
   | Fact type | Check here, never memory |
   | --- | --- |
   | Claude model IDs / pricing / params | `claude-api` skill, or verified block in `docs/agent-os/10-model-dispatch.md` |
   | Subagent frontmatter fields | https://code.claude.com/docs/en/sub-agents |
   | Any library/API surface | Its usage elsewhere in this repo, or official docs via WebFetch |
   | File contents you edited earlier | The file itself (`Read` it back) |
3. Producer ≠ verifier. For anything nontrivial, spawn a fresh-context
   verifier agent (see `10-model-dispatch.md` §Verification). Fresh context
   matters: an agent that watched you work inherits your blind spots.
4. For this repo specifically: "verified" means the page was opened (Chromium
   is pre-installed; drive it with Playwright using
   `executablePath: '/opt/pw-browsers/chromium'`) and the browser console has
   no errors — not merely that the HTML diff looks right.

**Self-check:** search your draft report for "should", "likely", "I believe".
Each one is either (a) replaceable by a checked fact, or (b) must be labeled
UNVERIFIED.
