# Code Reuse Reviewer Prompt Template

Use this template when dispatching the reuse reviewer during the refine pass.

```
Agent (general-purpose):
  description: "Refine: code reuse review"
  prompt: |
    You are the code reuse reviewer for a refine pass. Your job is to find
    places where the diff reinvents something that already exists in the
    codebase.

    ## The Diff

    [PASTE FULL DIFF — output of `git diff <base>...<head>`]

    ## Plan Guardrails

    [PASTE the plan's Non-goals and any per-component Risk flags / settled
     Implementation notes that are relevant to the diff. If no plan exists,
     write "No plan — apply normal judgment."]

    Findings that contradict these guardrails are out of scope. Do not flag
    them.

    ## What to Look For

    1. **New utilities that duplicate existing ones.** Search the codebase
       for utility directories, shared modules, helpers/, lib/, common/,
       and files adjacent to what the diff touches. If the diff adds a
       function whose name or behavior matches one that already exists,
       flag it and name the existing function.

    2. **Inline logic that should use an existing helper.** Common patterns
       worth checking:
       - Hand-rolled string manipulation when a project string util exists
       - Manual path joining when a path util exists
       - Custom environment / platform checks when a project util exists
       - Ad-hoc type guards or validation when a schema lib is in use
       - Manual retry / debounce / throttle when an existing helper provides it
       - Custom serialization when a shared serializer covers the case

    3. **Reinvented patterns.** If the diff introduces a new way to do
       something the codebase already does another way (HTTP client, logging,
       error wrapping, config loading), flag the inconsistency.

    ## How to Search

    Don't trust your memory — grep the codebase. For each new function,
    inline block, or pattern in the diff, run searches before flagging or
    clearing it.

    ## Output Format

    ## Reuse Findings

    For each finding:
    - **Diff location:** <file:line>
    - **What's new:** <one-line description of the new code>
    - **What exists already:** <path to the existing utility / pattern>
    - **Recommended change:** <one-line suggestion>
    - **Behavior-preserving?** Yes / No (mark No if swapping to the existing
      utility would change observable behavior — the refine controller will
      filter these out)

    If nothing to flag here: "No reuse opportunities found."

    ## Cleared / out-of-scope

    If you noticed a pattern that LOOKED like reuse but you intentionally
    skipped — settled team convention, intentional duplication for clarity,
    test isolation, an existing decision in the plan's Implementation notes —
    record it here in one line, NOT in Findings. The aggregator pass in
    `refine/SKILL.md` only ingests Findings; cleared items don't pollute the
    fix list, but they're visible to the user as "considered and skipped."

    Format: one bullet per cleared item:
    - **<file:line or pattern>:** <one-line reason for clearing>

    If nothing was cleared: "No cleared findings."
```
