# AGENTS.md

This is the **agent router**. It tells you where to find everything you need to do your work.

When you begin a task, your first action is always to consult this file and load the docs that are relevant to what you are doing. Do not proceed without loading applicable docs first.

---

## How to Use This File

1. Read the task or spec you have been assigned.
2. Look up the relevant categories below.
3. Load each listed document **before** writing any code or plans.
4. If a document does not exist yet, flag it as a TODO for human review and proceed with what you have.

---

## Workflow Docs

These define how you operate. Always load `AGENT_LOOPS.md` at the start of every session.

| Document | Purpose |
|---|---|
| `AGENT_LOOPS.md` | The full Night Shift loop — your operating procedure |
| `REVIEW_PERSONAS.md` | Defines the six review agents and their responsibilities |
| `TODOS.md` | Active bugs and human-flagged tasks |

---

## Specs

Completed feature specs live in `./.planning/specs/`. 

- Files prefixed with `draft-` are **not ready** — skip them.
- All other `.md` files in this folder are fair game for implementation.
- Each spec is self-contained and describes the feature, edge cases, and constraints.

---

## Skill Docs

Skill docs define *how* to do specific types of work. Load the relevant ones before implementing.

> Note: These are not agent "skills" in the framework sense. They are knowledge documents that encode patterns, conventions, and best practices for this codebase.

| Document | When to Load |
|---|---|
<!-- | `Docs/Skills/testing.md` | Any time you are writing or modifying tests |
| `Docs/Skills/api-design.md` | When adding or modifying API endpoints |
| `Docs/Skills/data-modeling.md` | When modifying database schema or data structures |
| `Docs/Skills/error-handling.md` | When implementing error paths or validation |
| `Docs/Skills/frontend-patterns.md` | When writing UI components or client-side logic |
| `Docs/Skills/performance.md` | When touching anything in a hot path | -->

*(Add new skill docs here as they are created. If you encounter a pattern that should be a skill doc, flag it in TODOS.md under NEEDS INPUT FROM USER.)*

---

## System Docs

System docs describe the architecture and subsystems of this codebase. Load the ones relevant to the area you are working in.

| Document | Covers |
|---|---|
| `./planning/codebase/ARCHITECTURE.md` | High-level system design, service boundaries, data flow |
| `./planning/codebase/STACK.md` | Languages, frameworks, and dependencies |
| `./planning/codebase/STRUCTURE.md` | Directory layout and key files |
| `./planning/codebase/CONVENTIONS.md` | Coding standards, naming patterns, and style rules |
| `./planning/codebase/TESTING.md` | Test setup, frameworks, and patterns |
| `./planning/codebase/INTEGRATIONS.md` | External services, APIs, and third-party dependencies |
| `./planning/codebase/CONCERNS.md` | Known tech debt, security issues, and risks |

*(Add new system docs here as the system grows.)*

---

## Review Personas Reference

The six review personas and which docs they own are defined in `REVIEW_PERSONAS.md`. Load that file before invoking any review agent.

Quick reference:

| Persona | Primary Concern |
|---|---|
| Designer | UX, interface consistency, user-facing behavior |
| Architect | System structure, service boundaries, long-term design |
| Domain Expert | Business logic correctness, domain rule compliance |
| Code Expert | Code quality, conventions, maintainability |
| Performance Expert | Speed, memory, scalability |
| Human Advocate | Real-world usability, accessibility, plain human judgment |

---

## Docs Maintenance

You are expected to keep docs up to date as part of implementation. If a doc is wrong, outdated, or missing information relevant to your task:

- Update it as part of your implementation commit.
- Do not leave docs in a state you know to be inaccurate.
- Each review persona is also responsible for flagging doc gaps in their area during the review steps.
