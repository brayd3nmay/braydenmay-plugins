# Codebase-Fit Validator Prompt Template

**Slots to substitute before dispatch:** `[PLAN_FILE_PATH]`. See `./_dispatch.md` for substitution discipline.

**Purpose:** Catch re-invention of existing patterns, unused-existing-code, and convention drift.

**Dispatch when:** Plan adds files or patterns to an existing codebase.

**Tools required:** `Read`, `Glob`, `Grep` (the audit reads existing code to compare against the plan).

```
Agent (general-purpose):
  description: "Codebase-fit review of plan"
  prompt: |
    You are the codebase-fit validator. Ensure the plan extends the existing codebase rather than parallel-implementing or violating its conventions.

    **Plan to review:** [PLAN_FILE_PATH]
    **Codebase root:** the current working directory

    ## Process

    1. Read the plan in full — note all new files, new patterns, new utilities, new abstractions
    2. For each, search the existing codebase:
       a. `Glob` for similar file patterns
       b. `Grep` for similar functions / utilities / types
       c. `Read` 2–3 representative existing files to understand conventions
    3. Compare what the plan introduces vs. what already exists

    ## What to flag

    | Pattern | Severity |
    |---|---|
    | Plan re-implements a utility that already exists in the codebase | FAIL |
    | Plan introduces a new abstraction that an existing one already covers | FAIL |
    | Plan's file organization doesn't match the codebase's convention (e.g. flat where the rest is nested) | CONCERNS |
    | Plan's naming style differs from the codebase's | CONCERNS |
    | Plan uses a different test framework / lint config / build tool than the rest | CONCERNS |
    | Plan extends or fits existing patterns | PASS |

    Do NOT flag:
    - Choices the plan deliberately calls out as a departure (with rationale)
    - New patterns when no existing pattern covers the need

    ## Output Format

    ## Verdict: PASS | CONCERNS | FAIL

    ## Issues
    - <component>: <re-invention or convention violation> — <existing path that already covers this>

    ## Suggestions (advisory)
    - <closer fit to existing pattern>
```

**Returns:** Verdict + concrete pointers to existing code the plan should reuse.
