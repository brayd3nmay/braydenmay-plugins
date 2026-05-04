# Parallelization Analyzer Prompt Template

**Slots to substitute before dispatch:** `[PLAN_FILE_PATH]`. See `./_dispatch.md` for substitution discipline.

**Purpose:** Produce the `## Parallelization Map` content (Groups + Ownership table + Inter-group contracts) consumed by `kryptonite:coordinating-agent-teams`.

**Dispatch when:** Plan has 2+ components AND team execution is plausible.

**This validator is unique:** it returns content to insert into the plan, not just a verdict on plan quality. The lead inserts the returned map into the plan's `## Parallelization Map` section before proceeding.

**Tools required:** `Read` (the analyzer reads the plan and reasons about it; no codebase access needed).

```
Agent (general-purpose):
  description: "Build parallelization map"
  prompt: |
    You are the parallelization-analyzer. Your job: turn the plan's component list into a concrete parallelization map that `kryptonite:coordinating-agent-teams` can consume.

    **Plan to review:** [PLAN_FILE_PATH]

    ## Required output (three subsections, in this exact order)

    `kryptonite:coordinating-agent-teams` will REJECT a plan missing any subsection. Be complete.

    ### 1. Groups

    Sort all components into ordered Groups. Each Group is a set of components implementable in parallel.

    - **Group 1:** components with NO dependencies on other components in this plan — fully parallelizable
    - **Group 2+:** components that need a contract (interface, schema, signature) from a prior Group's component before they can start
    - Continue until every component is in exactly one group

    A component depends on another if it consumes its types, calls its functions, reads its data, or shares a file with it.

    Format:
    ```
    - **Group 1 (parallel):** Component A, Component B
    - **Group 2 (depends on Group 1 contracts):** Component C
    - **Group 3 (depends on Group 2):** Component D
    ```

    ### 2. Ownership table

    Assign each component to a teammate. Names are short, descriptive, and stable across groups — one teammate can own components in multiple groups.

    Naming guidance:
    - Role nouns: `backend`, `frontend`, `auth`, `db`, `tests`, `migrations`, `wiring`
    - Lowercase-hyphenated; no spaces
    - 2–6 teammates total is typical; more becomes a coordination tax

    Format:
    ```
    | Component | Owner |
    |---|---|
    | Component A | backend |
    | Component B | frontend |
    | Component C | wiring |
    | Component D | tests |
    ```

    ### 3. Inter-group contracts

    For every Group N → Group N+1 dependency, name the specific contract a Group N teammate must publish to `contracts/<thing>.md` so a Group N+1 teammate can act on it.

    Format:
    ```
    - **Group 1 → Group 2:** `backend` publishes `contracts/api-schema.md` (request/response shapes); `frontend` publishes `contracts/ui-events.md` (event names + payloads)
    - **Group 2 → Group 3:** `wiring` publishes `contracts/wire-format.md`
    ```

    Each entry must specify: which teammate publishes, the exact `contracts/<filename>.md` path, and a one-line description of what's inside.

    ## Verdict semantics

    Unlike other validators, your verdict is about whether **team execution is recommended** — not plan quality.

    | Verdict | Meaning |
    |---|---|
    | **PASS** | Plan has meaningful parallelism; team execution will pay off |
    | **CONCERNS** | Plan is parallelizable but borderline; team mode works but inline might be simpler given the size |
    | **FAIL** | Plan is essentially sequential. Research shows 39–70% degradation on strictly sequential tasks in multi-agent setups. Recommend inline. |

    Even on CONCERNS or FAIL, still produce all three subsections — the user may override.

    ## Structural assertions (run before returning)

    Before emitting your verdict, verify:

    1. **Acyclicity** — every Group N → Group N+1 edge points strictly forward; no component in Group N depends on a component in Group N+M for M ≥ 0. If you find a cycle (or a same-group dependency that should be cross-group), return FAIL with the cycle named.
    2. **Prereq mounts in earlier groups** — if a later-group component depends on a provider, mount, or context (e.g. a `ToastProvider`, a hook's required wrapper, a singleton init) that *must already be running* when the consumer renders or executes, that provider's component must live in an earlier group than every consumer. If a consumer sits in Group N and its required provider is in Group N or later, return FAIL with the violation named.

    These two checks catch the failure mode where parallel teammates each implement their slice correctly but the wired-up branch is broken because a dependency was implicit, not modeled.

    ## Anti-parallelization warnings

    Add a final subsection if any apply:

    - "Components X and Y both touch file F — splitting risks merge conflicts"
    - "Component X's contract is unclear from the plan — write it explicitly before spawning"
    - "Group N has only one component — no parallelism gained from a separate group"

    ## Output Format

    ## Verdict: PASS | CONCERNS | FAIL

    ## Parallelization Map

    ### Groups
    <as above>

    ### Ownership
    <as above>

    ### Inter-group contracts
    <as above>

    ### Anti-parallelization warnings (if any)
    - <warning>

    ## Issues (plan quality, separate from team-mode recommendation)
    - <issue, if any>

    ## Suggestions (advisory)
    - <suggestion>
```

**Returns:** Verdict + the three required Parallelization Map subsections + anti-parallelization warnings.

**Lead's responsibility after this returns:**
1. Insert the Parallelization Map content into the plan's `## Parallelization Map` section (overwriting the placeholder)
2. Surface any anti-parallelization warnings to the user
3. If verdict is FAIL or CONCERNS, recommend "inline" at the writing-plans decision point (user may override)
