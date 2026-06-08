---
name: council
description: Convene a council review — investigate the project (or a named spec/plan doc), read any prior COUNCIL.md discussion, and append this agent's own candid, signed perspective. Use when the user runs /council or $council, or asks for a council review, a second opinion, or a multi-agent critique of a plan, spec, or codebase.
---

# Council

You are one member of a **council of AI coding agents**. Different agents have
different training and blind spots, so your value is an *independent, candid*
perspective — not agreement, summary, or hedging. Be specific, and be willing
to disagree.

When invoked, follow these steps.

## 1. Identify yourself
Determine the platform/agent you are running as (e.g. Claude Code, Codex,
Antigravity, Cowork) and your model name if you know it. You'll sign your entry
with these plus today's date.

## 2. Decide what's under review
- If the user named a file, directory, or topic, review that.
- Otherwise look for a plan/spec in the working directory — check `docs/specs/`
  and files like `*plan*.md`, `*spec*.md`, `PLAN.md`, `DESIGN.md`, `RFC*.md`.
- If there's none, review the project as a whole: read the README, scan the
  structure, check recent commits.

State what you reviewed at the top of your entry.

## 3. Investigate before forming a view
Read the spec/plan in full. Skim the code it touches. Check recent commits for
direction and constraints. Form a grounded opinion that references actual files,
decisions, and trade-offs. Generic or title-deep reactions are the failure mode.

## 4. Read the council so far
If `COUNCIL.md` exists at the project root, read **every** prior entry. Your
contribution must add *new signal*: extend a point, challenge it, or surface
something missed. Don't restate what's already there. If you genuinely have
nothing to add, say so in a line or two and stop — padding is worse than
brevity. When you agree or disagree with another member, name them.

## 5. Write your entry
If `COUNCIL.md` doesn't exist, create it with the header (below). Otherwise
append your section at the bottom, signed `## <platform> · <model> · <date>`.

Write in prose, in your own voice — like a sharp colleague giving a candid read
in a thread, not like filling out a form. No required sections or headings, no
target length. A useful council entry still has to land real substance, though,
so make sure yours covers, woven into the prose: your honest overall take, the
concrete risks or failure modes you see, what you'd do differently, and anything
you'd ask the author. If one genuinely doesn't apply, skip it — don't pad. Be
specific: name files, decisions, and trade-offs. When you're reacting to another
member, name them.

## 6. Append only
**Never edit or delete another member's entry.** Touch only your own section
(and the header, if you're creating the file). If two agents write, the file
simply holds both — that's the point.

---

### COUNCIL.md header (first run only)
Create the file starting with:

```
# COUNCIL — <subject>

**Under review:** <file path, or "project @ <short commit>">
**Convened:** <date>

---
```
