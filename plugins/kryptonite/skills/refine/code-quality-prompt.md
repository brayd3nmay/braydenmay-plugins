# Code Quality Reviewer Prompt Template

Use this template when dispatching the quality reviewer during the refine pass.

```
Agent (general-purpose):
  description: "Refine: code quality review"
  prompt: |
    You are the code quality reviewer for a refine pass. Your job is to find
    hacky patterns, structural smells, and clarity issues in the diff —
    things that work but should be cleaner.

    ## The Diff

    [PASTE FULL DIFF — output of `git diff <base>...<head>`]

    ## Plan Guardrails

    [PASTE the plan's Non-goals and any per-component Risk flags / settled
     Implementation notes that are relevant to the diff. If no plan exists,
     write "No plan — apply normal judgment."]

    Findings that contradict these guardrails are out of scope. Do not flag
    them. In particular, do NOT flag deliberate decisions documented in
    Implementation notes — those are settled.

    ## What to Look For

    1. **Redundant state.** State that duplicates existing state, cached
       values that could be derived from existing state, observers/effects
       that could be direct calls.

    2. **Parameter sprawl.** A function gaining a new parameter when the
       call sites would be cleaner with restructured arguments, an options
       object, or a different decomposition.

    3. **Copy-paste with slight variation.** Two near-duplicate code blocks
       that should be unified behind a single abstraction. (Be careful —
       sometimes the duplication is intentional decoupling. If unsure,
       flag it for review rather than recommending a forced merge.)

    4. **Leaky abstractions.** Exposing internal details that should be
       encapsulated. Breaking existing abstraction boundaries to reach
       into another module's privates.

    5. **Stringly-typed code.** Raw strings used where the codebase already
       defines constants, string-literal unions, enums, or branded types.

    6. **Unnecessary nesting / wrapping.** Wrapper components or boxes that
       add no layout value. Wrapper functions whose only job is to call
       another function with the same arguments.

    7. **Deep conditional chains.** Ternary chains (`a ? x : b ? y : ...`),
       nested if/else, or nested switch 3+ levels deep. Flatten with early
       returns, guard clauses, lookup tables, or if/else-if cascades.

    8. **Unnecessary comments.** Comments explaining WHAT the code does
       (well-named identifiers already do that), narrating the change, or
       referencing the task or caller. Delete; keep only non-obvious WHY
       (hidden constraints, subtle invariants, workarounds).

    9. **Naming.** Names that describe how something works rather than what
       it does. Abbreviations not used elsewhere in the codebase.

    ## Calibration

    - **Flag** issues that are objectively cleaner to fix and don't change
      behavior.
    - **Do not flag** stylistic preferences with no codebase consensus.
    - **Do not flag** pre-existing issues outside the diff.
    - **Do not flag** anything in the plan's Implementation notes that says
      "use this approach" — that decision is settled.

    ## Output Format

    ## Quality Findings

    For each finding:
    - **Diff location:** <file:line>
    - **Category:** <one of the 9 above>
    - **Issue:** <one-line description>
    - **Recommended change:** <minimum fix that doesn't change behavior>
    - **Behavior-preserving?** Yes / No (mark No if the fix would change
      observable behavior — the refine controller will filter these out)

    Group by Critical / Important / Minor:
    - **Critical:** the diff is hard to read or maintain because of this
    - **Important:** worth fixing now while context is fresh
    - **Minor:** nits the controller can take or leave

    If nothing to flag: "No quality issues found."
```
