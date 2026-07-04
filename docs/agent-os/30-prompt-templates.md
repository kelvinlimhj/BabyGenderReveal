# 30 — Subagent Prompt Templates

Copy the template, fill every `<...>`, delete nothing else. Each already
embeds the report contract from `10-model-dispatch.md`. Remember: built-in
Explore/Plan agents don't see CLAUDE.md — templates therefore carry their
own context.

Common footer (already included in each template below, shown once here):

> Return conclusions + `file:line` citations only; artifacts >30 lines go to
> a file, return the path. Label anything unrun as NOT VERIFIED. End with
> exactly one line: `STATUS: done | blocked: <reason> | partial: <what remains>`.

---

## T1 — Search / locate  (agent: `scout`, or Explore for wide sweeps)

```
GOAL: Find <what> so that <why the caller needs it>.
CONTEXT: Repo is <one line, e.g. "two static HTML pages, Firebase backend,
  no build">. Relevant dirs: <paths or "unknown — sweep the repo">.
LOOK FOR: <symbols / strings / patterns / behaviors>. Also try variants:
  <plurals, renames, abbreviations you can think of>.
DONE MEANS:
- Every location listed as `file:line` + ≤5-line quote, OR
- A definitive "absent", with the exact patterns and paths you tried.
REPORT: findings grouped by file; then one paragraph: what this implies for
  the caller's goal. [common footer]
```

## T2 — Implementation  (agent: `worker`)

```
GOAL: <change> because <motivation — what breaks or improves>.
SCOPE FENCE: Only touch <files/dirs>. Do NOT touch <files, e.g. Firebase
  config block>. If the right fix lies outside the fence, STOP and report
  blocked with reasoning.
CONTEXT: <how the touched area works today, 2–4 lines; key file:line refs>.
DONE MEANS:
- <observable behavior 1 — e.g. "vote.html shows a confirmation toast after voting">
- <observable behavior 2>
MUST NOT: <break duplicate-vote guard / change visual theme / etc.>.
FLOOR (run before reporting): <exact command or action, e.g. "open page in
  Chromium, exercise the flow once, console clean">. Final verification is a
  separate verifier's job — don't claim beyond what you ran.
REPORT: what changed and why, per file; floor commands run + output summary.
  [common footer]
```

## T3 — Refactor  (agent: `worker`; batch tail may go to `scout`)

```
GOAL: Restructure <what> into <shape> because <maintainability/perf reason>.
  Behavior must be IDENTICAL — this is the prime directive; abort any step
  that forces a behavior change and report it instead.
SCOPE FENCE: <files>. MUST NOT: alter observable behavior, public names
  used elsewhere, or <specifics>.
RECIPE (if pattern already proven): <steps>. Otherwise: propose the recipe
  on ONE instance first, report, and wait — do not batch-apply unproven.
DONE MEANS:
- Old pattern gone: `grep <old>` → 0 hits (show it).
- <behavior check identical, e.g. same page renders, console clean>.
- Diff contains no opportunistic extra changes.
REPORT: recipe used; instances changed (count + list); floor evidence.
  [common footer]
```

## T4 — Research  (agent: `worker`; web-heavy sweeps may use `scout` first)

```
QUESTION: <precise question>. DECISION IT FEEDS: <what will be chosen based
  on the answer — lets you judge relevance>.
SOURCES: Prefer <official docs / this repo / claude-api skill>. Training
  memory is NOT a source for: model IDs, API params, versions, prices —
  mark any such unfetchable fact UNVERIFIED rather than recalling it.
DONE MEANS:
- Direct answer in ≤5 sentences, THEN supporting detail.
- Every load-bearing fact has a source (URL or file:line) and fetch date.
- Explicit list: what you could NOT confirm.
REPORT: answer → evidence table (claim | source | date) → unconfirmed list.
  Full notes >30 lines go to a scratchpad file. [common footer]
```

## T5 — Review  (agent: `verifier` for verification; `worker` for opinionated review)

```
REVIEW TARGET: <diff / files / PR>.
YOU DID NOT WRITE THIS. Do not trust its comments or commit message; judge
  only what the code does.
CHECK, IN ORDER:
1. Correctness: does it do <intended behavior>? Trace the actual flow.
2. Breakage: does it violate any of <MUST NOT list from the original task>?
3. Repo fit: matches surrounding style/idiom? No opportunistic drive-by edits?
4. One thing the author forgot (adjacent behavior, edge case, leftover debug).
DONE MEANS: every item above has a verdict + evidence (`file:line` or command
  output). "Looks good" without evidence violates the contract.
REPORT: verdict per item; findings ranked by severity; ALSO NOTED section
  for #4. [common footer]
```
