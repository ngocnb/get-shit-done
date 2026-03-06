---
name: gsd:execute-tdd
description: Execute only the first (Wave 0) plan in a phase — writes failing test stubs and stops so you can implement the code yourself
argument-hint: "<phase-number>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
Execute only the Wave 0 (TDD stub) plan for a phase. Writes failing test stubs (RED phase), commits them, then stops — you implement the code yourself.

When you're done implementing, run /gsd:execute-phase {phase} to execute the remaining plans (which will pick up from where this left off, skipping the already-completed stub plan).
</objective>

<execution_context>
@./.claude/get-shit-done/workflows/execute-tdd.md
@./.claude/get-shit-done/references/ui-brand.md
</execution_context>

<context>
Phase: $ARGUMENTS

Context files are resolved inside the workflow via `gsd-tools init execute-phase` and per-subagent `<files_to_read>` blocks.
</context>

<process>
Execute the execute-tdd workflow from @./.claude/get-shit-done/workflows/execute-tdd.md end-to-end.
Stop after the stub plan is complete — do NOT run remaining waves or phase verification.
</process>
