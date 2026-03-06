<purpose>
Execute only the first plan (Wave 0) in a phase — the TDD test plan. Writes comprehensive failing tests (normal, abnormal, boundary cases) and stops. User implements the code themselves, then runs /gsd:execute-phase to complete remaining waves.
</purpose>

<required_reading>
Read STATE.md before any operation to load project context.
</required_reading>

<process>

<step name="initialize" priority="first">
Load all context in one call:

```bash
INIT=$(node "./.claude/get-shit-done/bin/gsd-tools.cjs" init execute-phase "${PHASE_ARG}")
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Parse JSON for: `executor_model`, `phase_found`, `phase_dir`, `phase_number`, `phase_name`, `plans`, `incomplete_plans`, `plan_count`.

**If `phase_found` is false:** Error — phase directory not found.
**If `plan_count` is 0:** Error — no plans found in phase.
</step>

<step name="find_first_plan">
Load the plan index to find Wave 0 (the test stub plan):

```bash
PLAN_INDEX=$(node "./.claude/get-shit-done/bin/gsd-tools.cjs" phase-plan-index "${PHASE_NUMBER}")
```

Parse JSON for `plans[]`. Find the plan with the lowest wave number that has `has_summary: false`.

**If all plans have summaries:** All plans already executed — report and exit.
**If the lowest-wave incomplete plan is NOT wave 0 or wave 1:** Warn that the TDD stub plan may already be done; confirm with user before proceeding.

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► EXECUTE TDD — PHASE {X}: {Name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Executing: {plan_id} — {plan_objective}

Writing comprehensive failing tests (normal, abnormal, boundary, security).
You implement the code. Then: /gsd:execute-phase {X}
```
</step>

<step name="execute_stub_plan">
Spawn gsd-executor for the first incomplete plan only:

```
Task(
  subagent_type="gsd-executor",
  model="{executor_model}",
  prompt="
    <objective>
    Execute plan {plan_number} of phase {phase_number}-{phase_name}.
    This is the TDD test-writing phase — write comprehensive failing tests. Do NOT implement the production code.

    Specifically:
    1. Install any missing test dependencies (e.g. new packages in pyproject.toml/package.json)
    2. Read the plan's <behavior> section thoroughly to understand every requirement
    3. Write stub source files — minimum code skeletons so tests can import and fail cleanly:
       - Python: functions/classes with correct signatures, `raise NotImplementedError` as body
       - TypeScript/JS: functions/classes with correct signatures, `throw new Error("not implemented")` as body
       - Only create the files and symbols the tests will import — no logic, no real implementation
       - Use correct type hints / TypeScript types from the plan's interfaces if specified
    4. Write COMPREHENSIVE tests — go beyond the plan's listed stubs. For every behavior, add:
       - Normal cases: valid inputs producing expected outputs
       - Abnormal cases: invalid inputs, error conditions, rejected requests
       - Boundary cases: empty values, min/max limits, edge values, off-by-one conditions
       - Security cases (if relevant): e.g. fake solved=true with wrong moves, unauthenticated access
    5. Each test must have a descriptive name that reads as a behavior spec (e.g. test_returns_400_for_illegal_move, not test_invalid)
    6. Run the full test file to confirm ALL tests FAIL for the right reason (NotImplementedError / "not implemented" — not ImportError or syntax errors)
    7. Fix any issues until tests fail cleanly on the stub (RED state with meaningful errors)
    8. Commit stubs and tests together: test({phase}-{plan}): add stubs and comprehensive failing tests for {feature}
    9. Create SUMMARY.md listing: stub files created + every test written grouped by: normal / abnormal / boundary / security
    10. Update STATE.md and ROADMAP.md

    Do NOT write real implementation logic. Stubs must raise NotImplementedError only. Stop after tests are confirmed RED.
    </objective>

    <execution_context>
    @./.claude/get-shit-done/workflows/execute-plan.md
    @./.claude/get-shit-done/templates/summary.md
    @./.claude/get-shit-done/references/tdd.md
    </execution_context>

    <files_to_read>
    Read these files at execution start using the Read tool:
    - {phase_dir}/{plan_file} (Plan)
    - .planning/STATE.md (State)
    - .planning/config.json (Config, if exists)
    - ./CLAUDE.md (Project instructions, if exists)
    </files_to_read>

    <success_criteria>
    - [ ] Test dependencies installed (if any)
    - [ ] Stub source files created with correct signatures and NotImplementedError bodies
    - [ ] Test file created with normal + abnormal + boundary + security cases
    - [ ] Every test has a descriptive behavior-spec name
    - [ ] Tests fail on NotImplementedError (not ImportError or syntax error)
    - [ ] Stubs and tests committed together
    - [ ] SUMMARY.md lists stub files + all tests grouped by category
    - [ ] STATE.md updated
    - [ ] ROADMAP.md updated
    </success_criteria>
  "
)
```

Wait for agent to complete.
</step>

<step name="verify_red_state">
Spot-check that tests exist and are failing:

```bash
# Verify SUMMARY.md was created
ls "{phase_dir}/{plan_id}-SUMMARY.md"

# Verify a test commit exists
git log --oneline --grep="{phase}-{plan}" | head -3

# Run tests to confirm RED (failure is expected and correct)
```

Read SUMMARY.md — check it mentions tests are in RED state.

**If SUMMARY.md missing or commit missing:** Report failure, ask user: "Retry?" or "Skip?"

**If tests are GREEN (passing) after stub plan:** This means the feature already exists or the stubs are wrong. Report to user:
```
⚠️  Tests passed unexpectedly after stub plan.
    Either the feature already exists, or the stubs don't test the right behavior.
    Check: {test_file}
    Then run: /gsd:execute-phase {X} to continue
```
</step>

<step name="offer_next">
Display completion status and guide user to implement:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► TDD TESTS READY ✓ — PHASE {X}: {Name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Tests are RED — implement to make them GREEN.

**Stub plan:** {plan_id} complete
**Test file:** {test_file_path}
**Run tests:** {test_command from plan}

───────────────────────────────────────────────────────────────

## ▶ Your Turn

Implement the code to make the failing tests pass:

1. Write the implementation
2. Run the tests until GREEN
3. When all pass, continue with remaining plans:

`/gsd:execute-phase {X}`

<sub>`/clear` first → fresh context window</sub>

───────────────────────────────────────────────────────────────

**Remaining plans in this phase:**
{list remaining incomplete plans with objectives, skipping the stub plan}

───────────────────────────────────────────────────────────────
```
</step>

</process>

<context_efficiency>
Orchestrator: ~10% context. Single subagent: fresh 200k. Stops after Wave 0 — no phase verification, no roadmap completion.
</context_efficiency>
