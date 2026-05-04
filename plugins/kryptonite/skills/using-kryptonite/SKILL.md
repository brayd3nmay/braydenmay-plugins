---
name: using-kryptonite
description: Use when starting any conversation - establishes how to find and use kryptonite skills (including agent-team coordination); the SessionStart hook injects this body so the rules apply before ANY response including clarifying questions
---

<SUBAGENT-STOP>
If you were dispatched as a one-shot subagent to execute a specific task, skip this skill — do the task and return. (Long-lived teammates spawned by the coordinating-agent-teams skill follow this skill normally; their briefing carries the carve-outs they need.)
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply to what you are doing, you ABSOLUTELY MUST invoke the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE. YOU MUST USE IT.

This is not negotiable. This is not optional. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

## Platform

Kryptonite is Claude Code only — depends on `TeamCreate`, `TeamDelete`, `SendMessage`, `Agent` with `team_name`, `TaskOutput`, and `TaskStop`. Will fail or degrade silently on other harnesses.

## Skills at a Glance

Core workflow: **brainstorming → writing-plans → (executing-plans | coordinating-agent-teams) → refine → finishing-a-development-branch**.

| Skill | Use when |
|---|---|
| `brainstorming` | Any creative work (new feature, component, behavior change) — explores intent and produces the Brainstorming Handoff that writing-plans consumes. |
| `writing-plans` | Brainstorming has converged. Sets up the feature worktree, drafts the plan to `docs/plans/`, dispatches parallel validators, presents the team-or-inline decision. |
| `executing-plans` | User picked "inline" at the writing-plans decision point. Walks components in dependency order with verification gates. |
| `coordinating-agent-teams` | User picked "agent team" and the plan has a populated parallelization map. Spawns long-lived teammates in worktrees, reviews every done claim, runs refine via a teammate. |
| `refine` | Closing pass after implementation completes. Three parallel reviewers (reuse, quality, efficiency); structural changes only, no behavior changes. |
| `finishing-a-development-branch` | Implementation + refine done. Verifies clean+green, presents PR/merge/squash/hand-off, drives cleanup of worktrees, branches, and the plan doc. |
| `using-git-worktrees` | Called by writing-plans (and coordinating-agent-teams for per-teammate worktrees) to set up isolated workspaces. |
| `dispatching-parallel-agents` | Powers one-shot fan-outs: writing-plans validators, refine reviewers, ad-hoc parallel investigations inside teammates. |
| `test-driven-development` | Implementing any feature or bug fix — every component, before any production code. |
| `verification-before-completion` | About to claim work is complete, fixed, or passing — required before committing or handing off. |
| `systematic-debugging` | Any bug, test failure, or unexpected behavior — root cause before fixes. |

All skills are invoked via the Skill tool.

## Instruction Priority

User instructions always win:

1. User's explicit instructions (CLAUDE.md, direct requests)
2. Kryptonite skills (override defaults where they conflict)
3. Default system prompt

## How to Access Skills

Use the `Skill` tool. When you invoke a skill, its content is loaded and presented to you—follow it directly. Never use the Read tool on skill files.

## Skill Prefix Convention

**Use the `kryptonite:` prefix at action points only.** Action points are: `Skill` tool invocations, sub-skill directives ("REQUIRED SUB-SKILL: Use kryptonite:X"), teammate briefing instructions where the agent will invoke a skill, frontmatter `description` cross-references where another plugin might collide, and `Calls:` / `Called by:` / `Pairs with:` lines in Integration sections (which readers may follow).

Descriptive prose ("called by the executing-plans skill", "the brainstorming skill explores intent") uses bare names. The discriminator: would a reader/agent be about to *act on* this name? If yes, prefix; if no, bare.

# Using Skills

## The Rule

**Invoke relevant or requested skills BEFORE any response or action.** Even a 1% chance a skill might apply means that you should invoke the skill to check. If an invoked skill turns out to be wrong for the situation, you don't need to use it.

When invoking a skill: announce *"Using [skill] to [purpose]"*, create a TodoWrite entry per checklist item if the skill has one, then follow the skill. About to EnterPlanMode without having brainstormed yet? Invoke `kryptonite:brainstorming` first.

## Red Flags

These thoughts mean STOP—you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" | Questions are tasks. Check for skills. |
| "I need more context first" | Skill check comes BEFORE clarifying questions. |
| "Let me explore the codebase first" | Skills tell you HOW to explore. Check first. |
| "I can check git/files quickly" | Files lack conversation context. Check for skills. |
| "Let me gather information first" | Skills tell you HOW to gather information. |
| "This doesn't need a formal skill" | If a skill exists, use it. |
| "I remember this skill" | Skills evolve. Read current version. |
| "This doesn't count as a task" | Action = task. Check for skills. |
| "The skill is overkill" | Simple things become complex. Use it. |
| "I'll just do this one thing first" | Check BEFORE doing anything. |
| "This feels productive" | Undisciplined action wastes time. Skills prevent this. |
| "I know what that means" | Knowing the concept ≠ using the skill. Invoke it. |

## Skill Priority

When multiple skills could apply:

1. **Process skills first** (brainstorming, debugging) — determine HOW to approach the task
2. **Coordination skills second** (coordinating-agent-teams) — if parallelizable, decide team shape before diving in
3. **Implementation skills last** — guide execution

Example: "multi-module refactor" → brainstorming → writing-plans → coordinating-agent-teams or executing-plans.

## Skill Types

Each skill states whether to follow it rigidly (e.g. TDD, debugging — don't adapt away the discipline) or adapt its principles to context.

## User Instructions

Instructions say WHAT, not HOW. "Add X" or "Fix Y" doesn't mean skip workflows.
