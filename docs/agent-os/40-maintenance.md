# 40 — Maintenance Protocol for agent-os Files

Read before editing CLAUDE.md or anything in `docs/agent-os/`.

## Edit permissions by file

| File | Weak model may edit alone? |
| --- | --- |
| `docs/agent-os/LESSONS.md` | YES — append-only, format below. Never rewrite or delete entries. |
| Verified-facts block in `10-model-dispatch.md` | YES — only to update values re-verified against live sources (claude-api skill / official docs). Update the checked date. Cite the source in the commit message. |
| `30-prompt-templates.md` | YES — additive tweaks (new template, extra checklist line) that came from a logged lesson. Link the lesson in the commit message. |
| `00-diagnosis.md`, `20-judgment-rubrics.md`, dispatch ladder & role table in `10` | ASK USER FIRST. These encode the original high-level judgment; a cheaper model "simplifying" them is the main degradation path. Propose the diff, get approval. |
| `CLAUDE.md` | Project section: yes, keep factual. Hard rules & index table: ASK USER FIRST. Keep ≤150 lines always. |
| `90-letter.md` | Never edit — historical document. Corrections go in LESSONS.md. |
| `.claude/agents/*.md` | Additive rule tweaks yes; changing `model`/`tools` fields: ASK USER FIRST. |

Backup rule: before any non-append edit to an agent-os file, `git diff` must
cleanly show the change and the commit message must say what and why —
git history is the backup; no separate .bak files needed.

## Lesson format (append to `docs/agent-os/LESSONS.md`)

```
## 2026-07-04 | <task in 5 words> | severity: low|med|high
WENT WRONG: <1–2 sentences, concrete>
ROOT CAUSE: <which failure mode or rubric gap — cite 00/10/20 section if one applies>
RULE CHANGE: <none | proposed edit to <file> §<section> (quote the new line)>
```

Log a lesson when: a verifier caught something you'd have shipped; an
approach hit the 2-retry cap; the user corrected you; a protocol here gave
wrong guidance. Don't log routine first-try errors.

## Compaction

- LESSONS.md > 150 lines → distill: cluster recurring causes, propose rule
  changes to the owning files (via ASK USER for protected ones), then move
  distilled entries to `docs/agent-os/LESSONS-archive.md`. Never delete.
- Any single protocol file > 200 lines → propose a split; the CLAUDE.md
  index table must be updated in the same commit.
- Staleness sweep (any session, ~quarterly): re-verify the verified-facts
  block in `10-model-dispatch.md`; check that all paths in CLAUDE.md's index
  table still exist (`ls` each).

## Cross-repo note

These protocols live in this repo because it's where the session that wrote
them had write access. Everything except the "Project" section of CLAUDE.md
and repo-specific verification floors is portable: to reuse elsewhere, copy
`docs/agent-os/` + `.claude/agents/` and rewrite only the project-specific
lines (marked by mentions of vote.html/results.html/Firebase).
