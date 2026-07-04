# LESSONS — append-only failure log

Format and rules: `docs/agent-os/40-maintenance.md`. Newest entries at the
bottom. Never rewrite or delete an entry; corrections get a new entry.

---

## 2026-07-04 | bootstrap: agent-os created | severity: low
WENT WRONG: Nothing yet — seed entry demonstrating the format.
ROOT CAUSE: n/a
RULE CHANGE: none

## 2026-07-04 | custom agents not hot-loaded mid-session | severity: med
WENT WRONG: Spawning subagent_type `verifier` failed with "Agent type not found" in the same session that created `.claude/agents/verifier.md`.
ROOT CAUSE: Agent definitions are loaded at session start; files created mid-session are only available to FUTURE sessions. Not covered in 00/10.
RULE CHANGE: none (workaround: fall back to `general-purpose` with an explicit `model` and paste the agent's rules into the prompt; next session can use the named agent).

## 2026-07-04 | adversarial review caught spoiler-guard gap | severity: high
WENT WRONG: Initial protocols demanded evidence-based verification but never forbade printing vote tallies — a verifier following the contract could have spoiled the gender reveal.
ROOT CAUSE: Verification rules and domain-specific harm were written in separate files with no hard-rule link (90-letter had it; CLAUDE.md hard rules didn't).
RULE CHANGE: applied — CLAUDE.md hard rule 6 (SPOILER GUARD) + structural-evidence line in `.claude/agents/verifier.md`.
