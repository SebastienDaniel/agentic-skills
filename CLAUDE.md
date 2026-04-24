# Claude Code — Global Guidelines

This file is loaded at the start of every session. It defines hard rules, the agentic workflow, and coding standards.

---

# Agentic workflow

This environment uses a spec-driven agentic workflow. See `~/.claude/README.md` for the full guide.

## Skill map

| Skill | When to use |
|---|---|
| `/arch-create` | First time on a project — generates `.claude/docs/ARCHITECTURE.md` |
| `/scaffold` | First time on a project — builds the validation pipeline |
| `/task-brainstorm` | Exploring a vague idea — produces a PRD |
| `/task-planner` | Turning a PRD into an executable task file |
| `/task-runner` | Executing a task file item by item |
| `/task-review` | Parallel code review before a PR |
| `/task-cleanup` | Archiving a completed task and committing |
| `/introspect` | End-of-session debrief to improve guidelines and skills |
| `/arch-update` | Syncing `ARCHITECTURE.md` after structural changes |
| `/appsec` | Security audit or hardening |

## Key rules

- A PRD (`.claude/tasks/*.prd.md`) is required before running `/task-planner`.
- `ARCHITECTURE.md` must exist before running planning or execution skills. Run `/arch-create` if it is missing.
- Never run validation commands directly (lint, typecheck, tests). Always invoke the `validate-changes` skill or the `validator` agent so results stay structured.
- When validating specific categories after a fix, pass the step names (e.g., `"types lint"`) instead of running the full pipeline.

---

# Code style

Read and apply `~/.claude/docs/code-style.md`.

---

# Introspection

Read and follow all guidelines in `.claude/docs/INTROSPECTION.md` when it exists.
