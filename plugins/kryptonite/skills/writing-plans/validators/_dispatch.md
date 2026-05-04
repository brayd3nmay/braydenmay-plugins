# Validator Dispatch: Slot Substitution

Every validator prompt declares its slots in a `**Slots to substitute before dispatch:**` line. Resolve each slot to its real value before dispatching — a literal `[SLOT]` token reaching the agent is never caught downstream: the validator reasons about the token text and returns a verdict that looks coherent but isn't grounded.

For most validators this is just `[PLAN_FILE_PATH]` (absolute path to the plan doc). `scope-prompt.md` uses two more (`[USER_REQUEST_VERBATIM]`, `[BRAINSTORM_DECISIONS]`) — see its header for sources and formatting rules.

Before dispatch, scan the substituted prompt for any remaining `[A-Z_]+` tokens.
