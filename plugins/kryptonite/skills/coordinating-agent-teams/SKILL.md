---
name: coordinating-agent-teams
description: Use when writing-plans validation has cleared and the user has chosen "agent team" execution. Spawns teammates in worktrees per the plan's parallelization map; lead orchestrates groups, reviews every done claim, runs a refine pass, then dissolves the team.
---

# Coordinating Agent Teams

Sets up and runs a multi-agent implementation of a validated plan. The lead (you) is **purely supervisory** — never implements. Teammates implement in isolated worktrees, communicate via three channels, and dissolve when done.

<HARD-GATE>
Do NOT spawn a team until ALL of:
1. You are inside the feature worktree writing-plans created (its Step 1). That worktree's branch (`feature/<name>`) is now the integration branch.
2. Plan exists at `docs/plans/YYYY-MM-DD-<feature>.md` inside that worktree and validators have cleared (writing-plans complete)
3. Plan has a populated `## Parallelization Map` with **groups, ownership table, and inter-group contracts**
4. User has explicitly chosen "agent team" at the writing-plans decision point
</HARD-GATE>

**Validation lives in the main chat only — never re-validate inside the team.** Teammates implement; you validate.

**Announce at start:** "I'm using the coordinating-agent-teams skill to spin up the team."

## Preflight: tmux check

Long-lived team work benefits from tmux. The team may run for tens of minutes; tmux lets the user detach/reattach without losing state, gives them separate windows to monitor teammates while keeping the lead session in view, and preserves work if SSH drops or the laptop sleeps.

Before doing anything else (BEFORE worktree setup, BEFORE TeamCreate), run:

```bash
if [ "${KRYPTONITE_SKIP_TMUX_CHECK:-}" = "1" ]; then
  echo "tmux check skipped (KRYPTONITE_SKIP_TMUX_CHECK=1)"
elif [ -n "$TMUX" ]; then
  echo "in tmux"
else
  echo "NOT in tmux"
fi
```

Set `KRYPTONITE_SKIP_TMUX_CHECK=1` to skip this check permanently — for users who never use tmux.

**If `$TMUX` is set (output is `in tmux`):** the user is already inside a tmux session. Skip this entire preflight section silently and proceed to the next checklist item. Do NOT surface anything to the user, do NOT ask about starting a session, do NOT mention tmux at all. The whole point of this gate is to avoid asking when the answer is already yes.

**If output is `tmux check skipped`:** the user opted out via env var. Same as above — proceed silently.

**If — and only if — output is `NOT in tmux`**, surface this to the user and wait for confirmation:

> Heads up — you're about to spawn a long-lived agent team that may run for several minutes or longer. I noticed you're not in a tmux session.
>
> Strongly recommended: detach now, start tmux (`tmux new -s <feature>`), and restart this command from inside the session. tmux lets you detach/reattach without losing the team's state and gives you separate windows for monitoring teammates.
>
> Continue without tmux anyway?

Wait for an explicit "yes / continue" response before proceeding. Do NOT infer consent. If they want to switch to tmux, stop cleanly — no worktrees created yet, nothing to undo.

## What you (the lead) do and don't do

| You DO | You DO NOT |
|---|---|
| Set up integration branch + per-teammate worktrees | Write or edit any code in any worktree |
| Spawn teammates by group, in dependency order | Pick up unclaimed tasks (that's a coordination signal, not a free task) |
| Review **every** done claim before marking complete | Re-validate the plan (validation lived in writing-plans) |
| Receive `SendMessage` from teammates; arbitrate scope/plan questions | Run `kryptonite:refine` yourself (the refine-teammate runs it; you confirm) |
| Spawn the refine-teammate at the end and confirm its report | Auto-commit or push the integration branch into main (per global CLAUDE.md) |
| Run `TeamDelete` and clean up worktrees | Skip the contract checkpoint between groups |

## Checklist

Create todos for these items with `TodoWrite` directly — don't paraphrase, use the tool:

```
TodoWrite({ todos: [
  { content: "Verify the plan is team-ready", status: "pending", activeForm: "Verifying plan is team-ready" },
  { content: "Verify the integration worktree", status: "pending", activeForm: "Verifying integration worktree" },
  { content: "Initialize contracts/ directory", status: "pending", activeForm: "Initializing contracts/ directory" },
  { content: "Commit contracts/ to integration branch", status: "pending", activeForm: "Committing contracts/" },
  { content: "Create per-teammate worktrees", status: "pending", activeForm: "Creating per-teammate worktrees" },
  { content: "TeamCreate + spawn Group 1", status: "pending", activeForm: "Spawning Group 1" },
  { content: "Receive done claims, review, ack/rework", status: "pending", activeForm: "Reviewing done claims" },
  { content: "Verify Group N → N+1 contracts at boundary", status: "pending", activeForm: "Verifying inter-group contracts" },
  { content: "Spawn Group N+1; repeat until all groups done", status: "pending", activeForm: "Spawning next group" },
  { content: "Spawn refine-teammate", status: "pending", activeForm: "Spawning refine-teammate" },
  { content: "Confirm refine report; present integration diff", status: "pending", activeForm: "Reviewing refine + presenting diff" },
  { content: "TeamDelete + hand off to finishing-a-development-branch", status: "pending", activeForm: "Tearing down + handoff" },
]})
```

`TodoWrite` is the existing convention across this plugin — don't invent alternative names like `TaskCreate`. Complete the items in order:

1. **Verify the plan is team-ready** — parallelization map, groups, ownership table, inter-group contracts all populated
2. **Verify the integration worktree** — you should already be in the feature worktree writing-plans created (its Step 1). That worktree on `feature/<name>` is now the integration worktree; you do NOT create a new one. The plan doc is already committed there (writing-plans committed it at the decision point).
3. **Initialize the `contracts/` directory** on the integration branch — `mkdir -p contracts && touch contracts/.gitkeep` (use `.gitkeep`, not a placeholder README).
4. **Commit `contracts/` to the integration branch** — `git -C <integration-worktree> add contracts/.gitkeep && git -C <integration-worktree> commit -m "init: contracts dir"`. This puts the directory on the branch so per-teammate worktrees (created in step 5) inherit it. The plan doc is already on the branch from writing-plans.
5. **Create per-teammate worktrees** via `kryptonite:using-git-worktrees`, branched from `feature/<name>` (now they inherit plan + `contracts/`)
6. **`TeamCreate`** to establish the team, then **spawn Group 1** with `Agent` calls (one per teammate, in a single message, `run_in_background: true`)
7. **Receive done claims, review each, send ack or rework**
8. **At each Group N → N+1 boundary, verify all required contracts are present on integration**
9. **Spawn Group N+1**; repeat 7–8 until all groups done
10. **Spawn refine-teammate** to run the `kryptonite:refine` skill on a worktree off the integration branch
11. **Confirm refine report**, present integration diff to the user for review
12. **`TeamDelete`**, then hand off to `kryptonite:finishing-a-development-branch` for worktree, branch, and plan-doc cleanup (don't clean those up here — that skill owns them and handles both topologies)

## Process Flow

```dot
digraph coord_flow {
    "Verify plan is team-ready" [shape=box];
    "Verify integration worktree" [shape=box];
    "Init + commit contracts/ dir" [shape=box];
    "Create per-teammate worktrees" [shape=box];
    "TeamCreate" [shape=box];
    "Spawn Group N (parallel)" [shape=box];
    "Receive done claim" [shape=box];
    "Review diff + verification" [shape=box];
    "Accept?" [shape=diamond];
    "SendMessage: rework" [shape=box];
    "SendMessage: ack" [shape=box];
    "All Group N tasks done?" [shape=diamond];
    "All required contracts published?" [shape=diamond];
    "More groups?" [shape=diamond];
    "Spawn refine-teammate" [shape=box];
    "Review refine diff" [shape=box];
    "Present to user" [shape=box];
    "TeamDelete + cleanup" [shape=doublecircle];

    "Verify plan is team-ready" -> "Verify integration worktree";
    "Verify integration worktree" -> "Init + commit contracts/ dir";
    "Init + commit contracts/ dir" -> "Create per-teammate worktrees";
    "Create per-teammate worktrees" -> "TeamCreate";
    "TeamCreate" -> "Spawn Group N (parallel)";
    "Spawn Group N (parallel)" -> "Receive done claim";
    "Receive done claim" -> "Review diff + verification";
    "Review diff + verification" -> "Accept?";
    "Accept?" -> "SendMessage: rework" [label="no"];
    "SendMessage: rework" -> "Receive done claim";
    "Accept?" -> "SendMessage: ack" [label="yes"];
    "SendMessage: ack" -> "All Group N tasks done?";
    "All Group N tasks done?" -> "Receive done claim" [label="no"];
    "All Group N tasks done?" -> "All required contracts published?" [label="yes"];
    "All required contracts published?" -> "Receive done claim" [label="no — wait"];
    "All required contracts published?" -> "More groups?" [label="yes"];
    "More groups?" -> "Spawn Group N (parallel)" [label="yes (next group)"];
    "More groups?" -> "Spawn refine-teammate" [label="no"];
    "Spawn refine-teammate" -> "Review refine diff";
    "Review refine diff" -> "Present to user";
    "Present to user" -> "TeamDelete + cleanup";
}
```

## Pre-spawn validation

The plan must contain these three pieces. If any are missing, STOP and return to `kryptonite:writing-plans`.

```markdown
## Parallelization Map

### Groups
- **Group 1 (parallel):** Component A, Component B
- **Group 2 (depends on Group 1 contracts):** Component C
- **Group 3 (depends on Group 2):** Component D

### Ownership
| Component | Owner (teammate name) |
|---|---|
| Component A | backend |
| Component B | frontend |
| Component C | integration |
| Component D | tests |

### Inter-group contracts
- **Group 1 → Group 2:** `backend` publishes `contracts/api-schema.md`; `frontend` publishes `contracts/ui-events.md`
- **Group 2 → Group 3:** `integration` publishes `contracts/wire-format.md`
```

## Worktree and branch layout

The integration worktree already exists — writing-plans created it on `feature/<feature>` as its first step, and you're inside it now. You add per-teammate worktrees alongside it (defer placement to `kryptonite:using-git-worktrees`):

```
<worktree-dir>/<feature>/             # branch: feature/<feature>           ← integration worktree (exists; from writing-plans)
<worktree-dir>/<feature>-<owner1>/    # branch: feature/<feature>/<owner1>  ← teammate 1 (NEW)
<worktree-dir>/<feature>-<owner2>/    # branch: feature/<feature>/<owner2>  ← teammate 2 (NEW)
...
```

Every teammate branch is created off `feature/<feature>` AFTER `contracts/` is committed (checklist step 4) — that way each teammate worktree inherits both the plan doc (already on the branch from writing-plans) and the contracts directory. Per-teammate worktree directories use the same parent directory using-git-worktrees chose during writing-plans Step 1.

`contracts/` lives on the integration branch and is the **only** path teammates may write to outside their own worktree.

## Spawning teammates

Each teammate is **long-lived**, not a one-shot subagent. Use `Agent` with:

- `subagent_type: "general-purpose"`
- `team_name: "<owner-name>"` — makes them addressable via `SendMessage`
- `name: "<owner-name>"` — same name for human readability
- `run_in_background: true` — multiple teammates run concurrently while you interleave reviews
- Do NOT pass `isolation: "worktree"` — you've already created the worktree manually
- `prompt`: the briefing template below

**Team lifecycle:** there is exactly one team per run. `TeamCreate` (step 6 of the checklist) establishes it; every subsequent `Agent` call with a `team_name` — including teammates spawned later in dependent groups, and the `refine-teammate` at the end — joins that same team. `team_name` labels the teammate for SendMessage routing, not a separate team. `TeamDelete` (step 12) tears the whole team down once. Do NOT call `TeamCreate` again mid-run.

Spawn ALL teammates in a group **in a single message** (multiple `Agent` tool uses in parallel).

### Briefing preflight (before each spawn)

Before sending the briefing for any teammate, substitute these placeholders to concrete values and verify nothing literal remains:

- `<owner-name>` → the teammate's name from the ownership table
- `<absolute-path>` → the actual absolute path to that teammate's worktree, computed from the worktree-dir + branch (NOT a relative path; NOT the integration worktree's path; NOT a placeholder)
- `<feature>` → the feature name
- `<branch>` → the teammate's branch (e.g. `feature/<feature>/<owner-name>`)
- `<Component name>` lines under "Your tasks" → the components owned by this teammate per the ownership table
- `<integration-head-shorthash>` → the actual short hash of `feature/<feature>` at spawn time, computed via `git -C <integration-worktree> rev-parse --short feature/<feature>` from the integration worktree. This is the value the teammate compares their own `git rev-parse --short HEAD` against in the "Verify your starting point" section of the brief.
- `<component-risk-flags>` → the verbatim Risk flags entries from the plan for the components this teammate owns. If multiple components, list each with its component name as a header. If all components have `None`, omit the section entirely from the brief.

After substituting, scan the briefing body for any remaining `<…>` placeholder text — if any survives, fix it before sending. Briefings shipped with literal placeholders cause the teammate to either ask "what's my path?" (round-trip wasted) or, worse, write to the wrong directory. The scan covers ALL angle-bracket placeholders in the substitution list above (including `<integration-head-shorthash>`, `<component-risk-flags>`, and any future additions) — no per-placeholder enumeration in the scan itself; the scan is the contract.

### Briefing template

```markdown
You are the **<owner-name>** teammate on the **<feature>** team.

You are NOT a one-shot subagent. You are a long-lived implementation agent that works iteratively with the lead and your peers. You may receive multiple `SendMessage`s during your run; respond to each before continuing.

## Your worktree
`<absolute-path>` — `cd` here for ALL file operations. Do NOT touch any other worktree.

## Your branch
`feature/<feature>/<owner-name>` — commit here only. Do NOT push to integration or main.

## Verify your starting point

Your branch should be at integration HEAD `<integration-head-shorthash>`. Confirm before doing any other work:

```bash
cd <absolute-path>
git rev-parse --short HEAD
```

If the output is NOT `<integration-head-shorthash>`, STOP and `SendMessage` lead with this exact payload:

`INTEGRATION_HEAD_MISMATCH: expected <integration-head-shorthash>, got <actual-output>`

Do not start work. Wait for the lead's `RESUME_AT: <new-hash>` reply (lead may rebase you, re-spawn you, or abort the spawn if the brief was wrong) before doing anything else.

## Your tasks
Read `docs/plans/<feature>.md` in full before starting. From the ownership table, you own:
- <Component name>
- <Component name>

Components you don't own belong to other teammates — do not implement them.

## Risk flags for your components

<component-risk-flags>

Risk flags marked `destructive:` REQUIRE you to `SendMessage` the lead with the exact payload `DESTRUCTIVE_OP_CONFIRM_REQUEST: <one-line description of the op>` and wait for an `APPROVED: <op>` or `DENIED: <op>; <reason>` reply before executing. Risk flags marked `novel:` are signals to the reviewer; you implement normally but expect closer scrutiny in refine. Risk flags marked `assumption:` are invariants you must preserve — if your implementation invalidates an assumption, escalate via the **Plan-defect escalation** protocol below.

## How to communicate

| Need | Channel |
|---|---|
| Plan/scope question, "I think my task is wrong", blocked on lead, request to spawn a peer | `SendMessage` to lead |
| Technical question for another teammate ("what field name?", "ETA on contract X?", "rename Y?") | `SendMessage` directly to that teammate by name (see ownership table) |
| Publishing an interface contract for downstream teammates | Write to `contracts/<thing>.md` and notify per **Contract publishing protocol** below |

## When not to respond

If a `SendMessage` from the lead contains no question, no new task, no rework instruction, and no stop signal — just an acknowledgement of your last message — do NOT respond. An empty "Acknowledged. Standing by." reply burns a model turn for no information. Continue with your current task or remain idle. (The Done claim protocol below is unaffected — you still wait for the lead's ack there before taking the next task.)

**Plan-defect escalation:** if you believe the plan's design is wrong — a contract conflicts, a component is missing, a decision in the plan won't work in this codebase — `SendMessage` the lead with a one-paragraph "plan defect: <what>; <why it doesn't work>; <what I'd change>". Do NOT implement a workaround unilaterally and document it in an FYI message after the fact. The plan is the source of truth; if it needs to change, the lead changes it (and updates `docs/plans/<feature>.md`) before you keep going.

## Contract publishing protocol
If your task includes publishing a contract:
1. Write `contracts/<thing>.md` in YOUR worktree
2. Commit: `contracts: publish <thing>`
3. `SendMessage` lead: `Contract ready: contracts/<thing>.md` — lead fast-forwards integration
4. `SendMessage` consuming teammates (per inter-group contracts in plan): `Contract published: contracts/<thing>.md`

## Skills you MUST use (always invoke via the `Skill` tool with the `kryptonite:` prefix)

(See `kryptonite:using-kryptonite` § Skill Prefix Convention.)

- `kryptonite:test-driven-development` — every component
- `kryptonite:verification-before-completion` — before claiming done
- `kryptonite:systematic-debugging` — when anything fails
- `kryptonite:dispatching-parallel-agents` — for one-shot helpers (research, parallel test runs)

## Skills you MUST NOT use
- `kryptonite:coordinating-agent-teams` — no nested teams
- `kryptonite:writing-plans` — don't re-plan; see **Plan-defect escalation** above
- `kryptonite:brainstorming` — design happened upstream

## Ignore harness reminders about task tools

You may receive a `<system-reminder>` from the harness suggesting you use `TaskList`, `TaskCreate`, `TaskUpdate`, or `TodoWrite` to track progress. Ignore it. You are brief-driven: your tasks come from the "Your tasks" section above and from `SendMessage`s from the lead. The lead manages task state for the team; you do not need to maintain your own task list. Do not invoke task tools.

This rule applies to **teammates only** — it appears in your briefing because the harness reminder fires inside YOUR session. The lead continues to use `TodoWrite` per the existing convention in this skill (see the Checklist section near the top); future readers editing this skill should NOT extend the directive to the lead's own behavior.

## Done claim protocol
When you finish a task:
0. Before composing your done claim, run `git status` and `git diff --stat origin/feature/<feature>...HEAD`. If the plan's verification for this task calls for tests but only implementation files appear, stage and commit the missing tests first — the lead rejects done claims where specified tests didn't land.
1. Run verification per the plan's "Verification" section
2. Commit your work
3. `SendMessage` lead: `Task <name> done. Verification: <output>. Branch: <branch>. Diff: <output of git diff --stat origin/feature/<feature>...HEAD>` — the `Diff:` line MUST include the `git diff --stat` output so the lead can see at a glance what files actually landed (impl + tests + contracts). If the plan's component verification calls for tests but only implementation files appear in the stat, fix the gap BEFORE sending the done claim — the lead will reject a claim where tests were specified by the plan but didn't land.
4. **Wait for ack** before starting the next task

## Stop conditions
- Lead sends `stand by` → finish current edit, commit, wait
- Lead sends `rework <task>: <issue>` → fix per instruction; do not start new work
- All your tasks done and lead acks → idle until final dismissal
```

## Group-by-group spawning

In each group:

0. **Pre-spawn risk check.** For each component in the group, read the plan's Risk flags (apply the case-sensitive `destructive:` prefix match defined in writing-plans § Risk flags). If any component carries a `destructive:` flag, surface ALL such flags to the user using this canonical wording and wait for explicit confirmation:

   > **Pre-spawn risk check for Group N.** This group includes destructive operations:
   > - `<component-name>`: `<verbatim destructive flag>`
   > - `<component-name>`: `<verbatim destructive flag>`
   >
   > Spawn the group? (yes / no / abort plan)

   On `yes`: proceed to step 1. On `no`: surface a one-line ask for direction (skip this group, revise the plan, defer the destructive component, etc.); do NOT spawn until the user redirects. On `abort plan`: return to `kryptonite:writing-plans` / `kryptonite:brainstorming` as appropriate.
1. Spawn ALL teammates in the group in one message (parallel `Agent` calls, `run_in_background: true`)
2. Watch `SendMessage` and `TaskOutput` for activity
3. Process each done claim using the **Reviewing a done claim** protocol below
4. When all Group N tasks are acked AND all Group-N→N+1 contracts are present, spawn Group N+1

The research is clear: spawning Group N+1 before Group N publishes its contracts wastes compute and degrades quality. Wait for the checkpoint.

## Reviewing a done claim (lead protocol)

For **every** done claim:

1. `git -C <teammate-worktree> diff main...HEAD` — read the actual changes
2. Cross-check against the plan's component contract for that task
3. Verify the verification: did the teammate run what the plan said? Re-run the cheap parts yourself; don't trust pasted output blindly
4. **If the change crosses an integration boundary** (touches a published contract or a file another teammate also touched): re-read the contract and confirm consistency
5. Keep ack messages to one line. Do NOT recap the implementation back to the teammate; they wrote it. The shared task list is the system of record for "what was done"; the ack is a permission token, not a summary.
6. Decision:
   - **Accept** → `SendMessage` ack: `ack <task>; ok to take next task` → mark complete in shared task list
   - **Rework** → `SendMessage`: `<task>: <specific issue>; please fix and re-claim`
   - **Escalate** → if accept/rework is unclear, surface to user before responding

The lead is the bottleneck on done claims by design. Skipping review is the #1 multi-agent failure mode.

## Handling teammate escalations

Two structured escalations may arrive from teammates that you must recognize and handle. Both reply formats are short and unambiguous so the teammate can pattern-match without ambiguity.

| Inbound payload | Lead response |
|---|---|
| `INTEGRATION_HEAD_MISMATCH: expected <X>, got <Y>` | (a) Verify which value is correct: `git -C <integration-worktree> rev-parse --short feature/<feature>`. (b) If teammate's `<Y>` is wrong, `git -C <teammate-worktree> reset --hard <correct-hash>`, then `SendMessage` teammate the literal reply token they're awaiting: `RESUME_AT: <correct-hash>`. (c) If brief's `<X>` was wrong (teammate is correct), the integration moved between brief generation and spawn — re-issue the brief with the new hash via a fresh task. |
| `DESTRUCTIVE_OP_CONFIRM_REQUEST: <one-line description of the op>` | (a) Surface the request verbatim to the user with a one-line ask: `Teammate <owner> requests confirmation for: <op>. Approve?` (b) On user approval, `SendMessage` teammate: `APPROVED: <op>`. (c) On user denial, `SendMessage` teammate: `DENIED: <op>; <user's reasoning>`. The teammate must wait for either reply before proceeding; do not approve without surfacing to the user. |

## Inter-group checkpoint

Before spawning Group N+1, verify the contracts Group N owed are on integration:

```bash
git -C <integration-worktree> log --oneline -- contracts/
git -C <integration-worktree> ls-files contracts/
```

If a required contract is missing: do NOT spawn Group N+1. `SendMessage` the responsible teammate for status.

## The refine pass

When all groups are done:

1. Create a fresh worktree off integration: `<worktree-dir>/<feature>-refine/` on branch `feature/<feature>/refine`. **Run `git worktree add` from the repo root, not from inside the integration worktree** — otherwise the path is resolved relative to cwd and you create a nested `.worktrees/X/.worktrees/Y/` structure (this has bitten the previous super-zoom run, then was inherited by `refine2` because that one was spawned with cwd inside the nested worktree). Always:

   ```bash
   repo_root="$(git -C <integration-worktree> rev-parse --show-toplevel)"
   ( cd "$repo_root" && git worktree add "<worktree-dir>/<feature>-refine" -b "feature/<feature>/refine" feature/<feature> )
   ```

   Before creating, sanity-check that `<worktree-dir>/<feature>-refine` is NOT a path inside another active worktree (`git worktree list` shouldn't show any active worktree as a prefix of the proposed path). If it is, refuse and pick a sibling location.

2. Spawn `refine-teammate` (`Agent` with `team_name: "refine"`, `run_in_background: true`) with this briefing. The refine-teammate joins the existing team established at step 6 — do NOT call `TeamCreate` again:

   > Run the `kryptonite:refine` skill on this worktree. The skill dispatches three parallel reviewers (code reuse, code quality, efficiency) and applies the surviving findings. **Do NOT change behavior** — refine is structural only. Commit your changes and `SendMessage` lead a summary of what you changed and what you deliberately left alone.

3. When refine-teammate claims done: review its diff using the same done-claim protocol
4. Merge `feature/<feature>/refine` back into `feature/<feature>` (the integration branch)
5. `SendMessage` refine-teammate dismissal

## Wrap-up

After refine lands on integration:

1. Show the user:
   - Total integration diff: `git diff main...feature/<feature>`
   - Test status on integration
   - Per-teammate contribution summary
2. **Stop.** Per the user's CLAUDE.md, do NOT auto-commit, merge, or push. The user reviews and instructs.
3. Once the user signals satisfaction with the integration branch:
   - **`TeamDelete`** to dissolve the team — do this BEFORE handing off to the closing skill, so background teammates aren't running while you're walking through integration choices. **`TeamDelete` is the dismissal** — call it directly; do NOT send goodbye `SendMessage`s first (each one is a billed turn for the teammates). (Replaces the older `shutdown_request`/`shutdown_response` handshake — one tool call, zero billed turns.)
   - **Hand off to `kryptonite:finishing-a-development-branch`.** That skill detects the team topology (integration branch + per-teammate sub-branches + multiple worktrees), verifies every worktree is clean and the integration branch is green, presents integration options (PR / merge / squash / hand off), and drives cleanup of all worktrees, the per-teammate branches, the refine branch, and the plan doc — without auto-committing, auto-pushing, or auto-merging.

Don't drive integration or cleanup decisions from this skill — finishing-a-development-branch is the single closing skill that handles both inline and team topologies. Your job ends at `TeamDelete` + handoff.

## Failure modes and recovery

| Symptom | Diagnosis | Fix |
|---|---|---|
| Teammate silent for a long stretch | Hung or stuck on a tool call | `TaskOutput` to peek; if truly stuck, `TaskStop` and respawn from a fresh task with the same briefing |
| Two teammates touched the same file | Plan decomposition was wrong | Pause both (`SendMessage stand by`), surface to user, revise plan, then resume |
| Done claim's verification doesn't match the diff | Fabricated success | Rework: send the exact verification command; require its real output |
| Contract published but consumer produces incompatible code | Contract was ambiguous | Plan defect. Pause both, fix the contract on integration, re-spawn the consumer's task with the corrected contract |
| Lead inbox is overwhelming | Too many teammates or tasks too fine-grained | Note for the next plan's parallelization-analyzer pass; don't dissolve mid-run |
| Teammate tries to invoke coordinating-agent-teams or writing-plans | Briefing failure | Re-send briefing; if it persists, `TaskStop` and respawn |

## Anti-patterns

- **Lead implements** — even one edit. Once the lead implements, it loses overview. Unclaimed tasks are a coordination problem, not free tasks.
- **Skipping done-claim review** — silent regressions are the #1 multi-agent failure mode.
- **Spawning all groups at once** — research shows 39–70% degradation on sequentially-dependent tasks in multi-agent setups. Spawn by group with contract checkpoints.
- **Letting teammates push to main or integration** — teammates push to their own branch only. Lead controls integration.
- **Forgetting `TeamDelete`** — leaves teammates running and billing in the background.
- **Re-validating the plan inside the team** — validation belongs to writing-plans in the main chat. The team implements.

## Integration

**Called by:** `kryptonite:writing-plans` (decision point: "agent team")
**Calls:** `kryptonite:using-git-worktrees` (worktree placement), `kryptonite:refine` (run by refine-teammate before `TeamDelete`)
**Closing pass:** after `TeamDelete`, hand off to `kryptonite:finishing-a-development-branch` — it owns integration choice and cleanup for both inline and team topologies
**Teammates internally use:** `kryptonite:test-driven-development`, `kryptonite:verification-before-completion`, `kryptonite:systematic-debugging`, `kryptonite:dispatching-parallel-agents`
