# 20 — Judgment Rubrics

High-level judgment converted into checklists a Sonnet-class model can run.
Each rubric: test → one positive example (apply it) → one negative example
(do NOT apply it). When two rubrics conflict, "stop and ask" beats
"escalate" beats "retry".

---

## R1 — When to escalate to a bigger model

Escalate ONLY if all three hold:
- [ ] The failure is a reasoning gap (wrong approach, subtle logic), not a
      missing-context gap — you already fed the agent everything relevant.
- [ ] You have a failure trail to hand over (attempts, exact errors, current
      hypothesis).
- [ ] The ladder in `10-model-dispatch.md` says it's time (Haiku ×1 miss,
      Sonnet ×2 misses on the SAME subtask).

✅ APPLY: Sonnet twice produced a Firebase rules patch that passes its own
test but breaks anonymous voting in a way it can't explain. Trail exists,
context was complete → escalate to Opus with the trail.

❌ DON'T: Haiku scout returned "no matches" for a symbol you know exists.
That's a bad search pattern (missing context/skill misuse), not a capability
gap → fix the grep pattern, stay on Haiku. Escalating would just search
wrong, expensively.

## R2 — When it's actually done

All must be true before you say "done":
- [ ] Every acceptance criterion has evidence from THIS session (command +
      output, or verifier PASS), not inference.
- [ ] A fresh-context verifier confirmed it (for anything nontrivial).
- [ ] The "MUST NOT break" list was checked, not assumed.
- [ ] Work is committed — and pushed if the session is remote/ephemeral.
- [ ] Your report contains zero unlabeled "should/probably/likely".

✅ APPLY: Edited results.html animation; verifier opened the page in
Chromium, saw the tally render, console clean; pushed. Say "done".

❌ DON'T: "The diff looks correct and the logic is straightforward, so this
should work." That is a `partial:` report with `NOT VERIFIED` markers, not
done. Rounding this up to "done" is the #1 trust-destroying move.

## R3 — When to stop and ask the user

Ask (and STOP working on that thread) when any of:
- [ ] Two readings of the request lead to materially different deliverables,
      and picking wrong wastes more than the round-trip to ask.
- [ ] Next step is irreversible or outward-facing: force-push, delete,
      deploy, send email/comment, spend money, touch prod data (here:
      the live Firebase DB — real guests' votes).
- [ ] Completing the task requires secrets/access you don't have.
- [ ] You found the stated task is built on a wrong premise (asked to fix X,
      but X works and Y is broken).

Do NOT ask about: naming, file layout, which of two equivalent
implementations, formatting — decide, note the decision in your report.

✅ APPLY: "Clean up the votes data" could mean delete test votes or
restructure the schema; both touch live data → ask, with a one-paragraph
summary of the two readings and your recommendation.

❌ DON'T: "Should I name the helper formatTally or renderTally?" — decide
and move on. Asking this burns the user's attention budget, which is the
scarcest resource in the system.

## R4 — Wrong-direction signals: change approach instead of retrying

Any ONE of these means the current approach is the problem — do not retry a
third time (hard cap from `10-model-dispatch.md`):
- [ ] Each "fix" spawns a new error in a different place (whack-a-mole).
- [ ] You're adding special cases to force an expected result.
- [ ] The diff keeps growing but the failing criterion hasn't moved.
- [ ] You're about to weaken the verification ("skip that test for now",
      "that console error is probably unrelated") to get to green.
- [ ] You catch yourself re-running the same command hoping for different
      output.

On any signal: stop, write 3 lines — what I believe / what the evidence
says / cheapest experiment to tell them apart — then either run that
experiment or restate the approach from scratch.

✅ APPLY: Third CSS tweak to fix a layout bug, each moving the breakage to
another element → stop patching; diagnose the actual box model with one
experiment (inspect computed styles) before touching code again.

❌ DON'T: A test fails because of a typo in the test itself. That's not a
direction signal, it's a bug — fix the typo. R4 is about repeated
whole-approach failure, not the first honest error.

## R5 — Quality floor: minimum verification by change type

The floor is what you must run BEFORE handing to the verifier; the verifier
then re-checks independently. Never negotiate the floor down mid-task (see
R4 bullet 4).

| Change type | Minimum floor |
| --- | --- |
| Doc / protocol file | Read back in full; every `path`, tool name, model ID in it actually exists (grep/check each) |
| HTML/JS in this repo | Page opens in Chromium, zero console errors, the changed behavior exercised once |
| Anything touching Firebase config/rules | Floor + explicit user confirmation first (R3 — live data) |
| Batch mechanical edits | Spot-check 3 random instances + grep for the old pattern returning zero |
| Subagent/prompt/config changes | Spawn it once on a toy task; confirm it obeys its report contract |

✅ APPLY: After a batch rename across files: grep old name → 0 hits, read 3
random sites, page still opens clean. Floor met → send to verifier.

❌ DON'T: "It's just a docs change, no verification needed." Docs ARE the
product of this repo's agent-os; a wrong path or tool name in a protocol
file silently corrupts every future session. Read it back and check every
reference.
