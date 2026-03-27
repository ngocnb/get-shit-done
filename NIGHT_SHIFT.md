# Night Shift

This file is the entry point for autonomous agent work during the Night Shift.

---

## Starting the Night Shift

To begin autonomous work, the agent should:

1. Read `AGENTS.md` to understand the documentation structure
2. Read `AGENT_LOOPS.md` to understand the full workflow loop
3. Read `.planning/ROADMAP.md` to see available phases
4. Begin at **Step 0 — Prep** in `AGENT_LOOPS.md`

---

## Current State

- **Roadmap:** `.planning/ROADMAP.md` — contains all phases
- **State:** `.planning/STATE.md` — tracks current phase
- **Active Specs:** `.planning/specs/` — specs ready for implementation
- **Active Bugs:** `.planning/bugs/` — bug reports (priority over features)
- **Draft Specs:** Files prefixed with `draft-` are NOT ready — skip them

---

## Workflow Summary

```
Step 1: Scan specs → Register as phases in ROADMAP.md
Step 6: /gsd:plan-phase {N}
Step 9: /gsd:execute-phase {N}
Step 13: /gsd:verify-work {N}
Step 15: Update ROADMAP, STATE, PROJECT, codebase docs
Step 16: Loop to next phase
```

---

## When Complete

After all phases are complete, the agent writes a summary report and waits for human review.

---

## Human Instructions

To start the Night Shift, simply tell the agent:

> "Start the Night Shift"

The agent will load `AGENT_LOOPS.md` and begin autonomous execution.