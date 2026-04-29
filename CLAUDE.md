# Claude Code — Global Guidelines

This file is loaded at the start of every session. It defines hard rules, the agentic workflow, and coding standards.

---

# Agentic workflow

This environment uses a spec-driven agentic workflow. See `~/.claude/README.md` for the full guide.

## Skill map

| Skill | When to use |
|---|---|
| `/arch-create` | First time on a project — generates `.context/docs/ARCHITECTURE.md` |
| `/scaffold` | First time on a project — builds the validation pipeline |
| `/task-brainstorm` | Exploring a vague idea — produces a PRD |
| `/task-planner` | Turning a PRD into an executable task file |
| `/task-runner` | Executing a task file item by item |
| `/task-review` | Parallel code review before a PR |
| `/task-cleanup` | Archiving a completed task and committing |
| `/introspect` | End-of-session debrief to improve guidelines and skills |
| `/arch-update` | Syncing `ARCHITECTURE.md` after structural changes |

## Key rules

- A PRD (`.context/tasks/*.prd.md`) is required before running `/task-planner`.
- `ARCHITECTURE.md` must exist before running planning or execution skills. Run `/arch-create` if it is missing.
- Never run validation commands directly (lint, typecheck, tests). Always invoke the `validator` agent (or, in projects where `/scaffold` has been run, the `validate-changes` skill) so results stay structured.
- When validating specific categories after a fix, pass the step names (e.g., `"types lint"`) instead of running the full pipeline.

---

# Introspection

Read and follow all guidelines in `~/.claude/INTROSPECTION.md` and `.claude/INTROSPECTION.md` when they exist.

---

# Rules

All `.md` files in `~/.claude/rules/` and `.claude/rules/` are auto-loaded into every session by Claude Code (v2.0.64+). They are first-class context — no explicit reference needed. Use `paths:` frontmatter to scope a rule to specific file patterns; otherwise it loads globally.
