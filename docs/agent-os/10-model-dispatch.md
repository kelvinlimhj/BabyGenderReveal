# 10 — Model Dispatch Protocol

Read before spawning any subagent. Applies to every session regardless of
which model is driving the main conversation.

---

## Verified facts (checked 2026-07-04 — re-verify per §Staleness below)

- Current model IDs: `claude-fable-5`, `claude-opus-4-8`, `claude-sonnet-5`,
  `claude-haiku-4-5-20251001`. Source: harness system info of the session
  that wrote this file. For pricing/limits, use the `claude-api` skill —
  do not hardcode prices here.
- Agent tool `model` parameter accepts aliases: `sonnet`, `opus`, `haiku`,
  `fable`. Subagent frontmatter `model` additionally accepts full model IDs
  and `inherit` (the default).
- Subagent frontmatter `effort` accepts: `low`, `medium`, `high`, `xhigh`,
  `max`. UNCONFIRMED: which levels each model supports — docs only say
  "available levels depend on the model". If an effort value errors, drop
  the field and rely on the model choice alone.
- Model resolution order for a subagent: `CLAUDE_CODE_SUBAGENT_MODEL` env
  var → per-invocation `model` parameter → frontmatter `model` → the main
  conversation's model (when frontmatter is `inherit` or omitted).
- Built-in `Explore` and `Plan` subagents do NOT receive CLAUDE.md or git
  status. Custom agents in `.claude/agents/` DO. Consequence: any prompt to
  Explore/Plan must carry its own context.
- Source for frontmatter/behavior: https://code.claude.com/docs/en/sub-agents
- UNCONFIRMED: whether requests safety-routed to Opus 4.8 consume the same
  quota window. Test on the usage dashboard; do not assume either way.

**Staleness rule:** if today is more than ~3 months after the checked date
above, or any alias errors, re-verify via the `claude-api` skill and update
this block (allowed per `40-maintenance.md`).

## Role table — who does what

| Role | Model | Use for |
| --- | --- | --- |
| Commander | whatever runs the main conversation | Decomposing the task, writing subagent prompts, integrating conclusions, talking to the user. Does NOT do bulk reads, repo scans, web research, batch edits, or verification itself. |
| Scout | `haiku` | Fan-out search, file inventory, "where is X used", extracting matches from logs, batch-applying an ALREADY-PROVEN mechanical pattern. |
| Worker | `sonnet` | Default for everything with judgment: implementation, refactors, research synthesis, writing docs/tests. When in doubt, Sonnet. |
| Heavy | `opus` | Escalation target only (see ladder below), second opinions on high-risk judgment, genuinely ambiguous architecture calls. Never the default. |
| `fable` | — | Do not dispatch to it by default; treat as unavailable unless the user says otherwise. |

Predefined agents in `.claude/agents/`: `scout`, `worker`, `verifier` —
prefer them over ad-hoc `general-purpose` spawns so tool limits and report
contracts apply automatically.

**Don't over-delegate:** a single Grep or one small Read is cheaper done
directly than spawning an agent (each spawn starts cold). Delegate when the
work would flood your context (>3 files, unknown location, long logs, batch
edits), not to perform ceremony.

## Task handoff — every subagent prompt has three parts

1. **Goal + motivation** — what to do AND why/what it feeds into, so the
   agent can make sane micro-decisions. One short paragraph.
2. **Acceptance criteria** — observable conditions for "done", including
   what must NOT change. Bullets, checkable, no adjectives.
3. **Report format** — exactly what to send back (see contract below).

Also include: any context the agent can't see (remember Explore/Plan don't
get CLAUDE.md), and a scope fence ("do not touch files outside X").
Fill-in templates per task type: `docs/agent-os/30-prompt-templates.md`.

## Report contract (put this in every subagent prompt)

- Return conclusions and `file:line` citations only. Do not return file
  dumps, full diffs, or raw logs.
- Any artifact longer than ~30 lines: write it to a file (scratchpad for
  temporaries, repo for deliverables) and return the path.
- End with exactly one line: `STATUS: done | blocked: <reason> | partial: <what remains>`.
- If you could not verify something, say `NOT VERIFIED` next to that claim —
  never round up to success.

## Escalation / de-escalation ladder

- Haiku errs once on a subtask → redo on Sonnet. Don't debug Haiku output.
- Sonnet fails the SAME subtask twice → escalate to Opus, and hand over the
  full failure trail (what was tried, exact errors, current hypothesis) —
  not just the original prompt.
- Once the hard instance is cracked and a pattern/recipe exists → hand the
  recipe DOWN to Sonnet/Haiku for batch application. Don't keep the
  expensive model for the repetitive tail.
- Hard cap: 2 retry rounds per approach on any subtask. Then either change
  approach (see wrong-direction signals in `20-judgment-rubrics.md`) or
  surface the blocker to the user. Never loop a third time on the same idea.
- Escalation is for capability gaps, not missing information. If the agent
  failed because the prompt lacked context, fix the prompt and stay at the
  same tier — a bigger model with the same starvation fails the same way.

## Verification — never self-certified

- The agent that produced work never verifies its own work, and the
  commander doesn't eyeball-verify either. Spawn `verifier` (fresh context,
  read-only + run) with the acceptance criteria and no description of how
  the work was done — describe WHAT should be true, not what was changed,
  so the verifier can't just agree.
- Files/docs → read-back: verifier reads the actual files and confirms each
  criterion, citing `file:line`.
- Code → execution: run tests if they exist; otherwise actually drive the
  behavior (for this repo: open the page in Chromium, check the console).
- High-risk judgment (irreversible, user-facing, security) → second opinion
  from Opus, or generate 2–3 candidate answers and have a fresh agent pick
  with reasons. Disagreement between candidates = signal to slow down.
- A verifier that only says "looks good" has failed the contract; it must
  cite evidence per criterion.
