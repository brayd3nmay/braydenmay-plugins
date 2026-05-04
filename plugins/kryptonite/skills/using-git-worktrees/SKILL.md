---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# Using Git Worktrees

## Overview

Git worktrees create isolated workspaces sharing the same repository, allowing work on multiple branches simultaneously without switching.

**Core principle:** Systematic directory selection + safety verification = reliable isolation.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Directory Selection Process

Follow this priority order:

### 1. Check Existing Directories

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**If found:** Use that directory. If both exist, `.worktrees` wins.

### 2. Check CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking.

### 3. Ask User

If no directory exists and no CLAUDE.md preference:

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/kryptonite/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## Safety Verification

### For Project-Local Directories (.worktrees or worktrees)

**MUST verify directory is ignored before creating worktree:**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:**

Per Jesse's rule "Fix broken things immediately":
1. Add appropriate line to .gitignore
2. Commit the change
3. Proceed with worktree creation

**Why critical:** Prevents accidentally committing worktree contents to repository.

### For Global Directory (~/.config/kryptonite/worktrees)

No .gitignore verification needed - outside project entirely.

## Creation Steps

### 1. Detect Project Name

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. Create Worktree

#### 2a. Path-segment collision check

Before computing the path, refuse branch names that would collide with an existing worktree at a path-segment level. A branch named `feature/auth/storage` cannot coexist with an existing worktree at `.worktrees/feature/auth/` because git would have to treat the same directory as both a worktree and a parent of one — git refuses, and the failure mode is confusing.

```bash
# Split the proposed branch on "/" and walk parent paths under the worktree-dir.
# If any prefix is already a worktree directory, refuse and ask the user to pick a different name.
proposed_path="$LOCATION/$BRANCH_NAME"
parent="$LOCATION"
for segment in $(echo "$BRANCH_NAME" | tr '/' ' '); do
  parent="$parent/$segment"
  if [ "$parent" != "$proposed_path" ] && [ -d "$parent" ] && git worktree list --porcelain | grep -q "^worktree $(realpath "$parent" 2>/dev/null)$"; then
    echo "ERROR: branch '$BRANCH_NAME' collides with existing worktree at '$parent'."
    echo "Pick a different name (e.g. 'feature/auth-storage' instead of 'feature/auth/storage'),"
    echo "or remove the existing worktree first: git worktree remove '$parent'"
    exit 1
  fi
done
```

This refuses `feature/x/storage` when `feature/x` is already a worktree, but legitimately allows sibling names like `feature/auth-v2` and `feature/auth-v3` (they don't share a directory prefix).

#### 2b. Compute path and create

```bash
# Determine full path
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/kryptonite/worktrees/*)
    path="~/.config/kryptonite/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch. Always run from repo root (use `git -C` if calling
# from elsewhere) so the path is resolved against the repo root, NOT the current cwd —
# otherwise an invocation from inside another worktree creates nested paths like
# `.worktrees/X/.worktrees/Y/`.
repo_root="$(git rev-parse --show-toplevel)"
( cd "$repo_root" && git worktree add "$path" -b "$BRANCH_NAME" )
```

**Cwd hygiene:** do NOT `cd "$path"` and rely on shell state persisting across subsequent steps. Each later step (project setup, baseline test) must either use `git -C "$path" …` / absolute paths or wrap its own commands in a subshell `( cd "$path" && … )`. Bare `cd` between tool calls silently drifts when an intermediate command fails, and has caused near-miss commits to the wrong branch.

### 3. Run Project Setup

Auto-detect and run appropriate setup. All commands operate on `$path` via subshell or `-C`-style flags — never relying on a persisted `cd`:

```bash
# Node.js
[ -f "$path/package.json" ] && ( cd "$path" && npm install )

# Rust
[ -f "$path/Cargo.toml" ] && ( cd "$path" && cargo build )

# Python
[ -f "$path/requirements.txt" ] && ( cd "$path" && pip install -r requirements.txt )
[ -f "$path/pyproject.toml" ] && ( cd "$path" && poetry install )

# Go
[ -f "$path/go.mod" ] && ( cd "$path" && go mod download )
```

### 4. Verify Clean Baseline

Detect whether the project actually has a runnable test suite, then branch the report accordingly. The three outcomes (passed / failed / no suite) must be distinguishable — a project with no tests must NOT be reported as "passing baseline."

```bash
# Pick the project-appropriate command, but only if it's actually defined.
# Examples:
#   - Node.js: a "test" script in package.json
#   - Rust:    a Cargo.toml with [[test]] targets or `tests/` dir
#   - Python:  pytest collects > 0 tests
#   - Go:      `go test ./...` finds test files
test_cmd_exists=false
test_passed=false
# (set test_cmd_exists=true if a runnable suite is detected, then run it from the worktree)

if ! $test_cmd_exists; then
  echo "No test suite found — skipping baseline test verification"
elif $test_passed; then
  echo "Tests passing"
else
  echo "Tests failed — report failures and ask whether to proceed or investigate"
fi
```

**If tests fail:** Report failures, ask whether to proceed or investigate.

**If tests pass:** Report ready.

**If no test suite is found:** Report ready WITHOUT claiming success on tests. Do NOT emit "Build passes baseline" or any phrasing that conflates "no suite" with "passed."

### 5. Report Location

Pick the line that matches the actual outcome from step 4 — never combine them:

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

OR (when no suite exists):

```
Worktree ready at <full-path>
No test suite found — skipping baseline test verification
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check CLAUDE.md → Ask user |
| Directory not ignored | Add to .gitignore + commit |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

### Skipping ignore verification

- **Problem:** Worktree contents get tracked, pollute git status
- **Fix:** Always use `git check-ignore` before creating project-local worktree

### Assuming directory location

- **Problem:** Creates inconsistency, violates project conventions
- **Fix:** Follow priority: existing > CLAUDE.md > ask

### Proceeding with failing tests

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Report failures, get explicit permission to proceed

### Hardcoding setup commands

- **Problem:** Breaks on projects using different tools
- **Fix:** Auto-detect from project files (package.json, etc.)

## Example Workflow

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at <project-root>/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## Red Flags

**Never:**
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking
- Assume directory location when ambiguous
- Skip CLAUDE.md check

**Always:**
- Follow directory priority: existing > CLAUDE.md > ask
- Verify directory is ignored for project-local
- Auto-detect and run project setup
- Verify clean test baseline

## Integration

**Called by:**
- **kryptonite:writing-plans** — Step 1 of the plan-writing checklist creates the feature worktree on `feature/<name>`. Planning, execution, refine, and finishing all happen inside that worktree.
- **kryptonite:coordinating-agent-teams** — uses this skill again to layer per-teammate worktrees alongside the integration worktree (which writing-plans already created)
- Any skill needing an isolated workspace

`kryptonite:executing-plans` and `kryptonite:coordinating-agent-teams` do NOT create the feature worktree themselves — they verify they're inside the one writing-plans created and stop if they're not.

**Cleanup when work is complete:**
- `kryptonite:finishing-a-development-branch` owns final cleanup of the feature/integration worktree(s) and per-teammate worktrees
- For ad-hoc usage outside the kryptonite workflow: `git worktree remove <path>` to remove a worktree directory
- Per the user's global CLAUDE.md, do NOT auto-merge or push — let the user review the diff and decide
