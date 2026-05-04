---
name: executing-plans
description: Use when you have a written implementation plan to execute inline in this session, sequentially component-by-component with verification gates
---

# Executing Plans

## Overview

Load the plan, review it critically, then implement one component at a time in dependency order. Each component is a unit of work — read its contract, implement, run its Verification block, commit, move on.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

If the plan has a populated `## Parallelization Map`, that section is for team mode — ignore it here. Inline execution is sequential by definition; you'll walk components one at a time in dependency order.

## What the plan looks like

Plans from writing-plans are component-based. Each `## Component: <name>` section gives you:

- **Goal** — one-sentence intent
- **Public interface / contract** — inputs, outputs, side effects, error modes
- **Touches files** — paths to create or modify
- **Dependencies** — other components or services this consumes
- **Implementation notes** — prose direction (approach, libs, tricky bits, decisions already made)
- **Verification** — the PASS/FAIL signal: tests to run, commands to execute, output to check
- **Risk flags** — destructive ops, novel patterns, hidden assumptions

There are no bite-sized steps and no checkboxes. You're an implementer with judgment — the contract and notes tell you what; you decide how, within the constraints.

## The Process

### Step 1: Verify you're in the feature worktree

The writing-plans skill already created the feature worktree on `feature/<name>` as its first step. Confirm: `pwd` should show the worktree path; `git rev-parse --abbrev-ref HEAD` should show `feature/<name>`. The plan file at `docs/plans/<file>.md` should be present and committed (writing-plans committed it at the decision point) — `git status --porcelain` should be clean.

If you're not in the worktree (e.g. the user invoked executing-plans cold without writing-plans), stop and ask the user — don't unilaterally create one. The plan and any prior validator work live in writing-plans' worktree; recreating loses both.

### Step 2: Load and review the plan

1. Read the plan file end-to-end
2. Build a dependency-ordered list of components by reading each component's `Dependencies` field. A component with no dependencies (or only external deps) can run first; everything else waits for what it consumes.
3. Critical-review pass: check for contradictions between component contracts, missing pieces, or contracts you don't understand. **If you find issues, stop and raise them with the user before touching code.** A flawed plan implemented faithfully is still a flawed result.
4. If clean, create todos with one entry per component, in dependency order. Use `TodoWrite` directly — don't paraphrase the step into prose:

   ```
   TodoWrite({ todos: [
     { content: "<Component A name>", status: "pending", activeForm: "..." },
     { content: "<Component B name>", status: "pending", activeForm: "..." },
     { content: "<Component C name>", status: "pending", activeForm: "..." },
     // …one per component
   ]})
   ```

   `TodoWrite` is the correct tool name (consistent with coordinating-agent-teams and brainstorming). Do not invent alternative names like `TaskCreate`.

### Step 3: Execute components in dependency order

For each component:

1. Mark its todo as `in_progress`.
2. **Re-read the component section** in the plan. Hold the contract and implementation notes in mind for this slice of work.
3. **Honor risk flags.** If the section flags a destructive op (deletion, schema change, mass rewrite), pause and confirm with the user before executing it. Risk flags are warnings the planner left for you on purpose.
4. **Implement.** Invoke `kryptonite:test-driven-development` and follow it for every component — RED, GREEN, REFACTOR. Inline execution does NOT relax TDD; the discipline is the same as in team mode. The only exceptions are the ones the test-driven-development skill itself names (throwaway prototypes, generated code, configuration files, no-test-framework projects), and they require asking the user first.
5. **Run the component's Verification block.** This is the gate — not your gut. If it specifies a test command, run it and confirm green. If it specifies an output check, check the output. Never mark a component done on the basis of "looks right."
6. If verification fails: debug it. Use `kryptonite:systematic-debugging`. Don't move on with a red component.
7. **Plan-vs-impl drift gate (run BEFORE marking the component done).** Re-read the component's plan section side-by-side with the diff for this component (`git diff --stat` for the file list, then targeted reads for the substantive changes). Confirm the diff matches the contract: same files touched, same public interface, no scope creep, no missing pieces. If the diff materially diverges from the plan — extra files modified, contract changed, behavior different from the Implementation notes — STOP. Surface the divergence to the user with a one-paragraph "here's what the plan said vs. what landed" summary; either revise the plan to match reality (and update `docs/plans/<file>.md`) or rework the implementation to match the plan. Do NOT silently let the plan and the diff disagree. Drift caught here is cheap; drift caught at refine or PR time is expensive.
8. Commit the component's work as a focused commit (per user's CLAUDE.md, do NOT auto-commit unless the user has asked you to commit during this session — otherwise stage the changes and ask the user "does this look good — ready for the next component?", not "can I commit?").
9. Mark the todo as `completed`.

### Step 4: Final verification

After every component is implemented and individually verified:

1. Run the full test suite on the feature branch. Confirm green.
2. Run any project-level checks the plan specifies under a top-level Verification section (lint, typecheck, integration tests).
3. Show the user:
   - Total diff vs main (`git diff main...HEAD`)
   - Test status
   - Commit list (`git log main..HEAD --oneline`)
4. **Stop and let the user review.** Per the user's CLAUDE.md, do NOT auto-commit, push, or merge into main — the user reviews and instructs.

### Step 5: Refine pass

Once the user has reviewed and signaled satisfaction with the implementation, run the `kryptonite:refine` skill on the feature branch. Refine is structural-only (no behavior changes); it tightens reuse, quality, and efficiency before the work goes anywhere.

If refine produces changes, re-run the full test suite to confirm the branch is still green.

### Step 6: Close out the branch

Hand off to `kryptonite:finishing-a-development-branch`. That skill verifies clean+green state, presents integration options (PR / merge / squash / hand off), executes the user's choice without auto-committing or auto-pushing, and drives cleanup of the worktree, the feature branch, and the plan doc.

Don't try to drive integration or cleanup yourself from this skill — delegate.

## When to Stop and Ask for Help

**STOP executing immediately when:**
- A component's contract is ambiguous and you'd be guessing
- A component's Verification block is missing or unclear (you can't tell when it's done)
- Two components' contracts contradict each other
- A risk flag fires and the user hasn't confirmed
- You hit a blocker that systematic-debugging doesn't resolve in a reasonable time

**Ask for clarification rather than guessing.** A wrong implementation forced through is more expensive than a question.

## When to Revisit Earlier Steps

**Return to Review (Step 2) when:**
- Implementing one component reveals that an earlier component's contract was wrong
- The user updates the plan based on findings
- The fundamental approach needs rethinking

If you have to revise a contract during implementation, **update the plan doc** before continuing, so the spec stays the source of truth.

## Remember

- Components, not steps — each `## Component: <name>` is one slice of work
- Dependency order, not document order — read `Dependencies` and topologically sort
- TDD every component — the test-driven-development skill is required, not optional
- Verification is the gate, not your judgment
- Honor risk flags — they were planted on purpose
- Stop when blocked, don't guess
- Never start implementation on main/master without explicit user consent

## Integration

**Required workflow skills:**
- **kryptonite:writing-plans** — creates the feature worktree and the component-based plan this skill executes
- **kryptonite:test-driven-development** — every component, no exceptions outside the ones the TDD skill itself names
- **kryptonite:systematic-debugging** — when verification fails
- **kryptonite:verification-before-completion** — when claiming work is done

**Closing pass (in order):**
- **refine** — structural cleanup once the user has reviewed; no behavior changes
- **finishing-a-development-branch** — verifies clean+green, presents integration options, drives cleanup

**Alternative workflow:**
- **coordinating-agent-teams** — better when the plan has a populated parallelization map (long-lived teammates in worktrees)
