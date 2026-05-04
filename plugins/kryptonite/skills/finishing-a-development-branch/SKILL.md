---
name: finishing-a-development-branch
description: Use when implementation and refine are complete and you need to close the loop — verifies green state, presents structured integration options (PR, merge, squash, hand off), and drives cleanup of worktrees, branches, and the plan doc. Never auto-commits, auto-pushes, or auto-merges.
---

# Finishing a Development Branch

## Overview

The closing pass for kryptonite work. After execution and `kryptonite:refine` land on the working branch, this skill verifies the branch is in a presentable state, lays out integration options, and walks the user through whichever option they pick — without ever committing, pushing, or merging on their behalf.

**Core principle:** Verify clean → Verify green → Present options → User picks → Execute the user-instructed steps → Clean up.

**Announce at start:** "I'm using the finishing-a-development-branch skill to close out this work."

## Two topologies

This skill handles both kryptonite execution paths:

- **Inline topology** — single feature branch (likely in a worktree), produced by `kryptonite:executing-plans`.
- **Team topology** — an integration branch with per-teammate sub-branches in their own worktrees, produced by `kryptonite:coordinating-agent-teams`. The integration branch is what gets shipped; the per-teammate branches are intermediate.

Detect which you're in before Step 3:

```bash
git worktree list
git branch --list 'feature/*' 'feature/*/*'
```

If you see `feature/<name>/<owner1>`, `feature/<name>/<owner2>`, etc., plus `feature/<name>` — that's team topology. The branch to ship is `feature/<name>` (integration). The per-teammate branches are cleanup candidates.

Otherwise, treat the current branch as the single shipping branch.

## The Process

### Step 1: Verify clean working state

For every worktree involved (just the current one in inline mode; integration + every teammate worktree in team mode):

```bash
git -C <worktree> status --porcelain
```

**If anything is uncommitted or untracked:** stop. Report it to the user. Per the user's global rule, do NOT run `git add` or `git commit` to "clean it up" — show them the dirty state and let them decide.

```
Worktree <path> has uncommitted changes:

<output>

Resolve before I can continue (commit, stash, or discard — your call).
```

Do not proceed to Step 2 until everything is clean.

### Step 2: Verify tests green on the shipping branch

The shipping branch is the current branch (inline) or the integration branch (team).

Run the project's full test command on it. Don't assume the verification blocks from individual components are sufficient — those verified slices, this verifies the whole.

```bash
# project test command — pick the right one
npm test      # or
pytest        # or
cargo test    # or
go test ./... # or whatever the project uses
```

Also run lint/typecheck if the project has them.

**If anything is red:**

```
Tests failing on <branch> (<N> failures):

<failures>

Cannot present integration options until the shipping branch is green. Want me to use systematic-debugging on this, or hand it back to executing-plans / coordinating-agent-teams?
```

Stop. Don't proceed to Step 3.

**If green:** continue.

### Step 3: Determine the base branch

```bash
# Try the obvious candidates
git merge-base HEAD main 2>/dev/null && echo "main" \
  || git merge-base HEAD master 2>/dev/null && echo "master"
```

If neither resolves, ask the user: "What's the base branch this should integrate into?"

Hold this as `<base>` for the rest of the skill.

### Step 4: Present integration options

Show the user a concise summary of what's about to ship, then the four options. Don't add commentary or recommend an option — let the user pick.

```
Shipping branch: <branch>
Diff vs <base>: <N> files changed, +<X> / -<Y>
Commits: git log <base>..<shipping-branch> --oneline → (paste the list)

Implementation complete and green. How do you want to integrate?

1. Open a Pull Request (push the branch and create a PR via gh)
2. Merge locally into <base> (no PR; you handle the push)
3. Squash-merge locally into <base> as a single commit (no PR; you handle the push)
4. Hand off — leave the branch as-is, I'll clean up worktrees only

Which option?
```

In team mode, also note: "Per-teammate branches (`feature/<name>/<owner>`) are intermediate and will be deleted in cleanup unless you say otherwise."

Wait for the user's choice. Do not infer from prior context.

### Step 5: Execute the chosen option (without committing/pushing on their behalf)

Per the user's global CLAUDE.md, **you do not run `git commit`, `git add`, `git push`, or anything that mutates remote state on your own**. The user said "do option N" — that's permission to set up the option, not blanket permission to push or merge. For each option, do the local-only mechanical work and then hand the user the exact commands to run themselves.

#### Option 1 — Open a Pull Request

**Before pushing, clean up the plan doc.** The plan doc is a working artifact, not part of what's shipping — PRs shouldn't include it. Show the user:

> The plan doc `docs/plans/<filename>.md` is committed on this branch. PRs shouldn't ship working artifacts — I'll delete it locally now, then you commit the deletion before we push. OK?
>
> (If you'd rather keep the plan doc as part of the PR's historical record, say so and I'll skip the deletion.)

If they confirm deletion:

```bash
rm docs/plans/<filename>.md
```

Stop. The user runs `git add -A && git commit -m "remove plan doc"` themselves (per the global no-auto-commit rule). Wait for them to confirm the deletion is committed before continuing.

If they decline (want the plan doc archived in the PR record): note it and proceed — the plan doc will be part of the PR.

1. Confirm the branch has an upstream tracking remote configured. If not, surface the command the user should run:
   ```bash
   git push -u origin <shipping-branch>
   ```
   Wait for them to push (or to explicitly tell you to push it).
2. Once pushed, draft the PR title and body from the commits and the plan doc. Show the user the draft. Don't open the PR until they say "open it" or paste the `gh pr create` command back.
3. When they confirm:
   ```bash
   gh pr create --title "<title>" --body "$(cat <<'EOF'
   ## Summary
   <2–3 bullets pulled from the plan's Goal + key components>

   ## Test plan
   - [ ] <verification commands the user should run on the PR>
   EOF
   )"
   ```
4. Capture the PR URL and report it back.

#### Option 2 — Merge locally into base

Show the user the exact sequence. Do not run it.

```bash
git checkout <base>
git pull
git merge --no-ff <shipping-branch>   # or --ff-only if their convention is linear
# run tests on the merged result before pushing
<test command>
# if green, the user pushes when they're ready:
git push
```

If they say "go ahead and run it," run it step by step, stopping after the merge and after the test run for confirmation. Never push without an explicit "push it now."

#### Option 3 — Squash-merge locally into base

Same shape as Option 2, but:

```bash
git checkout <base>
git pull
git merge --squash <shipping-branch>
# now the changes are staged on <base> as one diff
# the user reviews `git diff --staged`, edits the message, and commits themselves
```

Stop after the squash. Per the global rule, the user runs `git commit` and `git push` themselves.

#### Option 4 — Hand off

No git operations on the shipping branch. Skip to Step 6 (worktree/branch cleanup), but **do not delete the shipping branch or its worktree** — just clean up the intermediate stuff (per-teammate branches/worktrees in team mode).

### Step 6: Cleanup

Cleanup depends on which option was chosen.

#### Worktrees

Inline mode: if a worktree was created for this work, ask the user before removing it.

```bash
git worktree list
git worktree remove <worktree-path>     # only after confirmation
```

Team mode:
- Always offer to remove every per-teammate worktree (they served their purpose).
- The integration worktree: remove for Options 1 (PR is open elsewhere), 2, 3. For Option 4, ask.

```bash
# example
for owner in <owner1> <owner2> <owner3>; do
  git worktree remove <worktree-dir>/<feature>-<owner>
done
git worktree remove <worktree-dir>/<feature>-integration   # Options 1/2/3 only, after confirm
```

Confirm the list with the user before running any `git worktree remove` calls.

#### Per-teammate branches (team mode only)

The per-teammate branches are intermediate — their commits already live on the integration branch via fast-forwards. Offer to delete them:

```bash
for owner in <owner1> <owner2> <owner3>; do
  git branch -D feature/<feature>/<owner>
done
```

Use `-D` (force) because they were never merged through normal channels — their content is on integration via the lead's fast-forwards. Confirm with the user before running.

#### Refine branch (team mode only)

If `kryptonite:coordinating-agent-teams` created `feature/<feature>/refine`, it should already be merged into integration. Delete it:

```bash
git branch -d feature/<feature>/refine   # -d: safe; will refuse if unmerged
```

#### Plan doc

The plan doc is a working artifact of one feature's implementation, not a permanent record (see "Plan-doc lifecycle" in `kryptonite:writing-plans`).

- **Option 1 (PR)** — already handled inside Option 1's flow (deleted before PR creation, unless the user explicitly opted to keep it in the PR record). Skip this subsection.
- **Options 2, 3, 4** — delete the plan doc here, once integration is decided:

  ```bash
  rm docs/plans/<YYYY-MM-DD-feature>.md
  ```

  Show the user the path you're about to delete and wait for their go-ahead before running the command. The user commits the deletion themselves (per the global no-auto-commit rule). Users who want to keep plan docs archived can decline the deletion here — respect their answer either way.

#### Shipping branch

- **Option 1 (PR):** Keep until the PR merges. Don't delete.
- **Option 2/3 (local merge/squash):** After the user has merged and pushed `<base>`, offer to delete the shipping branch:
  ```bash
  git branch -d <shipping-branch>     # safe delete; refuses if unmerged
  ```
- **Option 4 (hand off):** Keep.

### Step 7: Final report

A short summary: what shipped, what's still pending (e.g. "user needs to push base"), what was cleaned up, what was preserved. Surface anything that requires the user's next action explicitly.

## Red Flags

**Never:**
- Run `git add`, `git commit`, `git push`, or `git merge` on the user's behalf without an explicit instruction in this turn — global CLAUDE.md rule, no exceptions
- Squash, force-push, or delete branches without confirmation
- Recommend an option in Step 4 — the user picks
- Skip Step 1 or Step 2 to "save time" — a dirty or red branch should never be presented for integration
- Delete worktrees containing uncommitted work
- Delete the plan doc before the user has decided how to integrate
- Treat option selection as blanket permission for everything that follows — re-confirm at push/merge/delete moments

**Always:**
- Verify clean state across all involved worktrees (every teammate worktree in team mode)
- Verify the full test suite green on the shipping branch
- Present exactly the four options, concisely
- Stop at every irreversible step and wait for explicit confirmation
- Leave the user the exact commands they'll run for `git commit` / `git push` themselves
- Surface what's still pending in the final report

## Quick reference

| Option | Local actions you may run | User must do themselves |
|---|---|---|
| 1. PR | Draft PR body, run `gh pr create` after explicit confirm | `git push -u origin <branch>` (unless explicitly delegated) |
| 2. Merge | `git checkout`, `git pull`, `git merge`, run tests | `git push`, any `git commit` if conflicts |
| 3. Squash | `git checkout`, `git pull`, `git merge --squash` | `git commit`, `git push` |
| 4. Hand off | No git ops on shipping branch | Everything |

## Integration

**Called by:**
- `kryptonite:executing-plans` — final step after the inline run + refine completes
- `kryptonite:coordinating-agent-teams` — drives the wrap-up after refine lands on integration

**Pairs with:**
- `kryptonite:using-git-worktrees` — cleans up worktrees that skill created
- `kryptonite:refine` — `kryptonite:refine` runs first; this skill assumes refine has already landed
