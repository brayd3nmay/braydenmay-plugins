# Kryptonite

A skill set for Claude Code agent teams. Kryptonite focuses on the workflow where a planner orchestrates one or more executor subagents through a complete feature — brainstorming, validated planning, execution, refinement, and clean handoff.

> **Claude Code only.** Kryptonite depends on `TeamCreate`, `TeamDelete`, `SendMessage`, `Agent` with `team_name`, `TaskOutput`, and `TaskStop`. It will fail or degrade silently on other harnesses.

## What's in the box

- **Component-based plan format.** Plans are organized into components (folders/modules) with tasks scoped to a single component, so executors can be dispatched one component at a time without crossing wires.
- **`coordinating-agent-teams` skill.** Explicit guidance for the planner role: how to dispatch, monitor, and integrate work from multiple long-lived teammate subagents.
- **`refine` skill.** A standalone reuse / quality / efficiency review pass over freshly-written code, separable from the plan/execute loop.
- **`finishing-a-development-branch` skill.** Drives the merge / PR / cleanup decision once implementation is verified.
- **SessionStart hook.** Surfaces the kryptonite skill set and the recommended workflow at the top of every new session. Workflow steps (brainstorm, write plan, execute, refine, finish branch) are invoked directly via the Skill tool — Claude auto-triggers them from natural-language prompts.
- **Shared validator-dispatch discipline.** Plan validators reference a single `_dispatch.md` doc spelling out slot-substitution rules — the lead reads it once before fanning out subagents, instead of re-deriving the pattern per validator. It's a discipline document, not an executable guard.
- **Brainstorming handoff capture.** The brainstorming skill writes a structured handoff document so the plan-writer skill picks up exactly what was decided, with no re-litigation.

## Workflow

The core loop is: **brainstorming → writing-plans → (executing-plans | coordinating-agent-teams) → refine → finishing-a-development-branch**.

For per-skill descriptions, see [`skills/using-kryptonite/SKILL.md`](skills/using-kryptonite/SKILL.md) — that file is the single source of truth and is injected at SessionStart so Claude has it in context. Don't duplicate the skill table here; update there instead.

## Installation

Kryptonite ships through the `braydenmay-plugins` marketplace.

In Claude Code, register the marketplace:

```
/plugin marketplace add braydenmay/braydenmay-plugins
```

Then install the plugin:

```
/plugin install kryptonite@braydenmay-plugins
```

To verify, start a new Claude Code session — the SessionStart hook should print the kryptonite skill list and recommended workflow.

## Configuration

Kryptonite reads the following environment variables. Set them in your shell rc, in your Claude Code `settings.json` `env` block, or wherever your harness exposes env vars to skill scripts.

| Variable | Values | Effect |
|---|---|---|
| `KRYPTONITE_SKIP_TMUX_CHECK` | `1` to enable | Skips the tmux preflight that `coordinating-agent-teams` runs before spawning a long-lived team. Use this if you never run Claude Code inside tmux and don't want the nag every time you start a team. |

Example — add to `~/.zshrc` (or equivalent) to disable the tmux check globally:

```bash
export KRYPTONITE_SKIP_TMUX_CHECK=1
```

Or scope it to a single Claude Code project via `.claude/settings.json`:

```json
{
  "env": {
    "KRYPTONITE_SKIP_TMUX_CHECK": "1"
  }
}
```

The check itself is documented in [`skills/coordinating-agent-teams/SKILL.md`](skills/coordinating-agent-teams/SKILL.md) under "Preflight: tmux check".

## License

MIT — see [`LICENSE`](LICENSE).
