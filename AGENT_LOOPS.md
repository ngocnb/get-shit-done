# AGENT_LOOPS.md

This document defines the automated workflow the agent follows during the **Night Shift** — the period when the human operator is away and the agent works autonomously through specs and bugs.

---

## Prerequisites Before Starting

- The human has placed completed spec documents in `./.planning/specs/`.
- Codebase documents are in `./.planning/codebase`. Read it first to understand the project structure.
- Specs prefixed with `draft-` are **ignored** — do not pick them up.
- The working tree should be on the correct branch before the loop begins.

---

## The Loop

### Step 0 — Prep

1. Inspect the current working tree.
2. If there are uncommitted changes:
   - If they are coherent and complete: **commit** them with a descriptive message.
   - If they are incomplete or unclear: **stash** them.
3. Run the **entire test suite**.
4. Fix any pre-existing test failures before proceeding.
5. Do not move forward until the baseline is clean and all tests pass.

---

### Step 1 — Select a Task and Register Phase

1. Run `/compact` to start with fresh context.
2. Reload ROADMAP.md and STATE.md to understand current state.
3. Check for **open bugs** first from `./.planning/bugs/`. Bugs have priority over features.
4. If no bugs remain, scan `./.planning/specs/` for non-draft specs.
5. For each new spec not yet in `ROADMAP.md`:
   - Read `.planning/ROADMAP.md` to find the next available phase number.
   - Add a new phase entry to ROADMAP.md with the spec name and path.
   - Update `.planning/STATE.md` to reference the new phase.
6. Pick the next **pending phase** from ROADMAP.md.
7. Work on one phase at a time. Do not parallelize across phases.

---

### Step 2 — Load and Analyze the Spec

- Read the full spec document for the selected task.
- Understand the feature, all described edge cases, and any constraints.
- Note anything unclear — do not invent requirements; flag gaps as TODOs for human review.

---

### Step 3 — Load Relevant Documentation and Code

- Consult `@AGENTS.md` to identify which docs apply to this task.
- Load all relevant workflow docs, skill docs, and system docs.
- Read the relevant source code to understand current implementation context.

---

### Step 4 — Develop Implementation Plan

- Determine the **phase number** from ROADMAP.md for the current spec.
- Capture context using `/gsd:discuss-phase {phase_number} --auto`.
- Build a detailed implementation plan using `/gsd:plan-phase {phase_number} --auto`.
- The planner automatically detects TDD candidates and creates appropriate plan types (`type: tdd` for test-driven features, `type: execute` for standard tasks).
- The human will **never read this**. It is for your own reasoning and structure.
- Be thorough. Consider the full impact on the system.

---

### Step 5 — Coverage Gate (MANDATORY)

**Cannot be skipped.** This catches the "spec says X, plan forgets X" problem.

The `/gsd:plan-phase` workflow includes a Context Decisions Coverage Gate that:
1. Extracts all decisions from CONTEXT.md `<decisions>` section
2. Verifies each decision appears in at least one plan task
3. Blocks proceeding if any decision is uncovered

**If gaps found:**
- Choose: Revise plans, remove decision from CONTEXT.md, or defer to future phase. If decision is removed or defer, move them to TODOS.md for the user to read it later.
- Loop back to Step 4 if revision needed
- Do NOT proceed to Step 6 until all decisions are covered

---

### Step 6 — Run Review Agents (Pre-Implementation)

**MANDATORY — cannot be skipped.**

- Invoke each of the six review personas defined in `REVIEW_PERSONAS.md`:
  - Designer
  - Architect
  - Domain Expert
  - Code Expert
  - Performance Expert
  - Human Advocate
- Each persona loads their assigned documentation and reviews the plan against it.
- Each persona also flags any gaps or outdated sections in their own docs.

---

### Step 7 — Adapt Plan Based on Reviews

**MANDATORY — cannot be skipped.**

- Incorporate feedback from all review agents.
- If any reviewer does not give a **green light**, revise the plan and **loop back to Step 4**.
- Continue until all six reviewers give a green light.

---

### Step 8 — Implement

- Execute the implementation plan using `/gsd:execute-phase {phase_number}`.
- Update all affected documentation in-place.
- Follow existing conventions exactly unless a doc explicitly states otherwise.

---

### Step 9 — Static Analysis and Tests

Run all of the following, fix any issues, and iterate:

- Type checking (strict mode)
- Linting (strict mode)
- Compiler / build
- Bundle size reporter (if applicable)
- Any other configured static analysis tools
- The relevant feature/bug tests (TDD tests are run by execute-phase)

Do not proceed until everything passes cleanly.

---

### Step 10 — Full Regression Suite

- Run the **entire test suite**.
- Fix any regressions introduced by this change.
- Do not skip or suppress failures.

---

### Step 11 — Run Review Agents (Post-Implementation)

- Run all six review agents again, this time against the **implementation diff**.
- If any reviewer does not give a green light, loop back to Step 9.
- Continue until all six reviewers approve the implementation.

---

### Step 12 — Verify Work

- Run `/gsd:verify-work {phase_number}` to perform user acceptance testing.
- This validates the implementation meets the spec's goals.
- If verification fails, loop back to Step 9 to fix issues.
- Do not proceed until verification passes.

---

### Step 13 — Capture Unrelated TODOs

- Note any unrelated issues, tech debt, or concerns noticed along the way.
- Add them to the `TODOS.md` file under a section marked **NEEDS INPUT FROM USER**.
- Do **not** fix unrelated issues during this task. Stay focused.

---

### Step 14 — Wrap Up the Phase

1. Write a `CHANGELOG` entry for this phase in the project root `CHANGELOG.md` file.
2. Commit with a **detailed commit message** written for human context:
   - What changed and why.
   - Any non-obvious decisions made.
   - References to the spec.
3. Move the spec file to `./.planning/specs-done/` folder. Bugs should be moved to `./.planning/bugs-done/` folder.
4. Update ROADMAP.md: mark the phase as `completed`.
5. Update STATE.md:
   - Set current phase to the next pending phase.
   - Update "Next Action" to point to the next phase.
6. Update PROJECT.md:
   - Add any new tech stack or integration info discovered.
   - Update key documents table if new docs were created.
7. Update `.planning/codebase/` docs if the implementation changed:
   - `ARCHITECTURE.md` — if system structure changed.
   - `STACK.md` — if new dependencies or providers added.
   - `INTEGRATIONS.md` — if new external services added.
   - `CONVENTIONS.md` — if new patterns established.
   - `CONCERNS.md` — if new tech debt or risks introduced.

---

### Step 15 — Loop

- Return to **Step 1** and select the next task.
- Continue until all bugs are resolved and all completed specs are implemented.

---

### Step 16 — Final Report

When all tasks are complete:

- Write a **concise summary report** for human review.
- Keep it brief — detailed context lives in the commit messages.
- Cover: what was completed, what was skipped and why, and anything requiring human input.

---

### Step 17 — Go Silent

- The Night Shift is complete.
- Do **not** start new work.
- Wait silently for the human to review in the morning.

---

## Important Principles

- **Never wait for human input** mid-loop. If blocked, log it as a TODO and move on.
- **Strictness is good.** Strict type checking and linting catch agent errors that would otherwise reach human review.
- **Tokens are cheap. Human time is not.** Run as many validation passes as needed before a human ever looks at it.
- **Do not fix unrelated things.** Capture them and move on.
- **Docs are living artifacts.** Update them as part of implementation, not after.
- **Run continuously until complete.** After each GSD skill returns, immediately invoke the next step. Do NOT stop at "Next Up" outputs. The loop only ends when Step 16 (Final Report) is reached.

---

## Continuous Execution Flow

When running Night Shift, execute this sequence without stopping:

```
Step 1: /clear → Reload ROADMAP.md, STATE.md
    ↓
Step 2-3: Load spec, documentation, and code (read files)
    ↓
Step 4: /gsd:plan-phase {N} --auto (planner handles TDD internally)
    ↓ (if context > 70%: /compact)
Step 5: Coverage Gate — verify all CONTEXT.md decisions covered
    ↓ (if gaps: loop to Step 4)
Step 6-7: Run review agents, adapt plan (MANDATORY — all 6 reviewers must pass)
    ↓
Step 8: /gsd:execute-phase {N} --auto (runs TDD and implementation plans)
    ↓ (if context > 70%: /compact)
Step 9-10: Static analysis, linting, full test suite
    ↓
Step 11: Run review agents (post-implementation)
    ↓
Step 12: /gsd:verify-work {N}
    ↓ (immediately after verification)
Step 14: Update docs, commit, move spec
    ↓ (immediately)
Step 15: Loop to Step 1 for next phase (with /compact)
```

**DO NOT** output "Next Up" blocks or wait for user input between steps.

**TDD is handled by GSD:** The planner detects TDD candidates and creates appropriate plan types. The executor runs TDD plans with RED→GREEN→REFACTOR cycles automatically. No separate test-writing step is needed.

---

## Context Management

**At Step 1 (start of each loop iteration):**
- Run `/compact` to start with fresh context
- Then reload: ROADMAP.md, STATE.md, and current phase plans

**When context exceeds 70% usage:**
- Run `/compact` automatically to compress conversation history
- Continue execution without stopping
