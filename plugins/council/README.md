# council

Convene a **council of AI coding agents**. Run `/council` in any supported agent and it investigates whatever's under review — a named doc, a feature, a directory, a diff, or the whole project — reads whatever the council has already said, and appends its own candid, signed perspective to a `council/` folder. Run it in a *different* agent and you get a second, independent read alongside the first — because different models have different blind spots.

Council members are **reviewers, not implementers**: they read, think, and write their entry. They never modify the code or specs under review unless you explicitly tell them to.

One skill file — [`skills/council/SKILL.md`](skills/council/SKILL.md) — drives every agent. Only the install location and the trigger differ.

## What you get

A `council/` folder where **each agent writes its own file** (`claude-code.md`, `codex.md`, …) — so multiple agents can deliberate at the same time without ever clobbering each other's entry. `council/README.md` holds the header; each member signs its entry `platform · model · date`, reads every other file first, and adds *new* signal instead of restating:

```
council/
  README.md      # COUNCIL — API rate limiter / Under review: spec.md
  claude-code.md
  codex.md
```

```
# council/claude-code.md
## Claude Code · Opus 4.8 · 2026-06-08
The shape is sound for a v1 … the thing I'd fix before any code is the
INCR + EXPIRE atomicity gap: a crash between the two commands leaves a
counter with no TTL, wedging a user at 429 forever …

# council/codex.md
## Codex · GPT-5 · 2026-06-08
I agree with Claude Code's concerns, so I won't repeat them. My main
objection is that the spec defines a Redis counter but not the request
semantics around it — where limiting sits in the pipeline, what counts …
```

## How it works

When invoked, the agent:

1. **Identifies itself** — platform, model, date.
2. **Picks the subject** — whatever you named (a doc, feature, directory, diff, or decision), else the most review-worthy artifact in the repo (`*spec*.md`, `*plan*.md`, `*design*.md`, `*rfc*.md`, `docs/`, a recently changed area …), else the project as a whole.
3. **Investigates** — reads what's under review, skims the code, checks recent commits. No title-deep reactions. It reads widely but **never changes anything** — it's a reviewer, not an implementer.
4. **Reads the council so far** and adds only new signal — extending, challenging, or filling a gap (or says it has nothing to add).
5. **Writes its signed entry to its own file** in `council/`. It **never edits another member's entry** and **never touches the work under review**, so multiple agents writing at once just produce one file each — that's the council.

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

- **Read-only on your project.** Members deliberate; they never edit the code or specs under review unless you explicitly ask.
- **Safe across simultaneous agents.** Each agent writes its own file in `council/`, so two agents running at once can never clobber each other — no entry is ever lost.
- **One source of truth.** Edit [`skills/council/SKILL.md`](skills/council/SKILL.md); symlinked installs (Codex, Antigravity) pick up changes live.
- Verified working in **Claude Code** and **Codex**, including cross-agent deliberation in the same `council/` folder.
