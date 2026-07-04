# 90 — Letter to Future Sessions

Written 2026-07-04 by the Fable 5 session that created this agent-os.
Historical document — do not edit (corrections go to LESSONS.md).

## Three things the user didn't ask for, but matter most

1. **The binding constraint is discipline, not intelligence.** A Sonnet that
   keeps its context clean, writes acceptance criteria first, and never
   self-certifies will outperform a bigger model that wanders. Nothing in
   these files requires cleverness; everything requires actually doing the
   checklist. When you're tempted to skip a step because the task "is
   simple", that temptation is exactly the failure mode the step exists for.
   Corollary: the user's attention is the scarcest resource in this system —
   spend it only where R3 says to.

2. **This machinery is portable, and also optional.** The protocols live
   here because this repo is where the authoring session had write access,
   but a two-file HTML project doesn't need a dispatch ladder. For a
   five-minute task, CLAUDE.md's five hard rules are the whole protocol —
   spawning three agents to change one CSS color is ceremony, and ceremony
   is how this system dies (see degradation below). The real payoff comes
   from copying `docs/agent-os/` + `.claude/agents/` into repos with real
   complexity and rewriting only the project-specific lines (see
   `40-maintenance.md` §Cross-repo note). Consider `~/.claude/agents/` for
   user-level reuse of scout/worker/verifier.

3. **This repo has a spoiler hazard and a live-data hazard.** The Firebase
   config in the HTML is public by design; the security rules are the only
   protection for real guests' votes — treat any rules/config change as a
   production change (R3 + R5). Subtler: this is a gender REVEAL. The
   parents and guests may not want to know the tally or the answer before
   the event. Never print database contents, vote counts, or anything
   implying the result into chat, commits, or logs unless the user
   explicitly asks for the numbers. An agent that helpfully pastes the DB
   state could ruin the surprise — a harm no test will catch.

## How this system will most likely degrade, and the prevention

- **Ceremony drift:** future sessions keep the rituals (STATUS lines, agent
  spawns) but drop the substance (evidence). A format-perfect report with no
  command output is the tell. Prevention: R2's "evidence from THIS session"
  test; verifiers must cite per-criterion evidence or they've failed.
- **Simplification erosion:** a session "tidies" the rubrics and deletes the
  ✅/❌ examples as redundant. The examples ARE the content — a weak model
  reads the rule through them. Prevention: the ASK-USER-FIRST list in
  `40-maintenance.md`; if you're reading this while planning such a cleanup,
  stop and ask.
- **Staleness cascade:** a model alias or doc URL rots, the first error
  convinces a weak session the whole system is broken, and it abandons all
  of it. Prevention: the staleness rule in `10-model-dispatch.md` — update
  the one dead fact, log a lesson, keep the rest.
- **LESSONS.md as write-only memory:** entries accumulate, nobody reads
  them, the same mistake repeats. Prevention: the CLAUDE.md trigger ("when a
  task feels familiar, check LESSONS") and the 150-line compaction rule.

## Honest confidence report on these deliverables

Lowest confidence first:

1. **The predefined agents were not test-spawned at write time.** The
   frontmatter follows docs verified the same day, but per R5's own floor
   ("spawn once on a toy task") they should be exercised — the closing
   review of the authoring session attempts this; if it didn't happen, do
   it in your session and log the result.
2. **The "top 3" in `00-diagnosis.md` is judgment, not telemetry.** It's
   drawn from how this harness generally fails, not from measurements of
   THIS user's sessions. The three are real, but their ranking — and
   whether something else belongs in the top 3 for this user — should be
   revised from LESSONS.md evidence after a few weeks of use.
3. **Haiku's capability boundary in the role table is an estimate.** Where
   exactly "mechanical" ends and "needs Sonnet" begins should be calibrated
   by experience; expect to move tasks between tiers and log it.
4. **Unconfirmed facts, flagged where used:** whether safety-routing to
   Opus 4.8 consumes the same quota window (test on the usage dashboard);
   which effort levels each model supports.
5. **Moderate-to-high confidence:** the dispatch ladder, report contract,
   rubrics R1–R5, and templates — these encode well-tested agent-management
   practice and user-specified policy, and they fail soft (worst case: some
   wasted spawns, never silent corruption).
