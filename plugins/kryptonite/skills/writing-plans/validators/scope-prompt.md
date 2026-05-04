# Scope Validator Prompt Template

**Slots to substitute before dispatch:** `[PLAN_FILE_PATH]`, `[USER_REQUEST_VERBATIM]`, `[BRAINSTORM_DECISIONS]`. See `./_dispatch.md` for substitution discipline.

**This validator has the most slots in the directory — substitute carefully.** A literal `[USER_REQUEST_VERBATIM]` token reaching the agent yields a verdict that reads coherent but is not grounded in the actual ask.

- `[USER_REQUEST_VERBATIM]` — paste the user's exact wording from the plan doc's `## Brainstorming Handoff` section ("User's original ask (verbatim)" subsection). Do not paraphrase.
- `[BRAINSTORM_DECISIONS]` — paste the structured bullet list from the plan doc's `## Brainstorming Handoff` section (decisions reached, alternatives rejected). If brainstorming did not run, write `(no brainstorming preceded this plan)` rather than leaving the slot empty.
- `[PLAN_FILE_PATH]` — absolute path to the plan doc.

**Purpose:** Confirm the plan implements what the user asked for — no scope creep, no missing requirements.

**Dispatch when:** Plan is non-trivial.

```
Agent (general-purpose):
  description: "Scope review of plan vs user request"
  prompt: |
    You are the scope validator. Verify the plan implements what the user asked for — nothing more, nothing less.

    **Plan to review:** [PLAN_FILE_PATH]
    **User's original request:** [USER_REQUEST_VERBATIM]
    **(If brainstorming preceded planning, also reference key decisions from that conversation — provided as [BRAINSTORM_DECISIONS].)**

    ## What to check

    | Question | Severity if "no" |
    |---|---|
    | Does every component in the plan trace back to something the user asked for or a decision made during brainstorming? | FAIL (scope creep) |
    | Does the plan cover every part of the user's request? | FAIL (missing requirement) |
    | Does the plan's "Non-goals" section explicitly call out things the user mentioned but doesn't want now? | CONCERNS if missing |
    | Are there features in the plan that the user didn't ask for and brainstorming didn't surface? | CONCERNS or FAIL depending on size |

    ## Calibration

    | Severity | Pattern |
    |---|---|
    | FAIL | Plan is implementing a different thing than what the user asked for, OR missing a core part of the ask |
    | CONCERNS | Minor scope expansion (one extra feature; one optional polish step) |
    | PASS | Plan matches the ask cleanly |

    Do NOT flag:
    - Style/wording preferences in how scope is stated
    - Items that brainstorming explicitly chose to defer (mention as suggestions, not issues)
    - Implicit features any reasonable interpretation of the request would include
    - Components added because a downstream validator (e.g. integration-contract) requires them

    ## Output Format

    ## Verdict: PASS | CONCERNS | FAIL

    ## Issues
    - <component>: <out-of-scope addition> OR <missing-from-plan piece of the ask>

    ## Suggestions (advisory)
    - <reframe scope here, e.g. "move X to Non-goals">
```

**Returns:** Verdict + scope deltas (additions and omissions).
