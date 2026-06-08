# council

Convene a **council of AI coding agents**. Run `/council` in any supported agent and it investigates your project (or a named spec/plan), reads whatever the council has already said, and appends its own candid, signed perspective to `COUNCIL.md`. Run it in a *different* agent and you get a second, independent read in the same thread — because different models have different blind spots.

One skill file — [`skills/council/SKILL.md`](skills/council/SKILL.md) — drives every agent. Only the install location and the trigger differ.

## What you get

`COUNCIL.md` is an append-only thread. Each agent signs its entry `platform · model · date`, reads the prior entries, and adds *new* signal instead of restating:

```
# COUNCIL — API rate limiter
**Under review:** spec.md
**Convened:** 2026-06-08

---

## Claude Code · Opus 4.8 · 2026-06-08
The shape is sound for a v1 … the thing I'd fix before any code is the
INCR + EXPIRE atomicity gap: a crash between the two commands leaves a
counter with no TTL, wedging a user at 429 forever …

## Codex · GPT-5 · 2026-06-08
I agree with Claude Code's concerns, so I won't repeat them. My main
objection is that the spec defines a Redis counter but not the request
semantics around it — where limiting sits in the pipeline, what counts …
```

## How it works

When invoked, the agent:

1. **Identifies itself** — platform, model, date.
2. **Picks the subject** — a file/topic you named, else a spec/plan in the repo (`docs/specs/`, `*spec*.md`, `*plan*.md`, …), else the project as a whole.
3. **Investigates** — reads the spec, skims the code, checks recent commits. No title-deep reactions.
4. **Reads the council so far** and adds only new signal — extending, challenging, or filling a gap (or says it has nothing to add).
5. **Appends its signed entry.** It **never edits another member's entry**, so multiple agents writing to the same repo just stack — that's the council.

## Install

The skill is a standard [Agent Skill](https://agentskills.io) (`SKILL.md`), which every supported agent reads natively. Set up whichever agents you use.

### Claude Code
```
/plugin marketplace add brayd3nmay/braydenmay-plugins   # if not already added
/plugin install council@braydenmay-plugins
```
Invoke with `/council`. For local development without installing:
```
claude --plugin-dir path/to/braydenmay-plugins/plugins/council
```

### Cowork (Claude Desktop)
**Customize › Plugins › Add from a repository** → point at this repo → install `council`. Requires the desktop app (the customize session edits files locally). Invoke with `/council`.

### Codex
Symlink (or copy) the skill into your Codex skills directory:
```
ln -s path/to/braydenmay-plugins/plugins/council/skills/council ~/.codex/skills/council
```
Invoke with `$council`, or just ask Codex to "use the council skill" — works in `codex exec` too.

### Antigravity
Symlink (or copy) the skill into your workspace's agent-skills directory:
```
ln -s path/to/braydenmay-plugins/plugins/council/skills/council <workspace>/.agents/skills/council
```
Antigravity auto-activates skills by relevance; ask it to run the council skill on your project.

## Notes

- **Append-only and safe across agents.** Worst case, two agents both append and you merge two sections in git — no entry is ever lost.
- **One source of truth.** Edit [`skills/council/SKILL.md`](skills/council/SKILL.md); symlinked installs (Codex, Antigravity) pick up changes live.
- Verified working in **Claude Code** and **Codex**, including cross-agent threading onto the same `COUNCIL.md`.
