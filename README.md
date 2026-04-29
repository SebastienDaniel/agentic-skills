# Agentic Spec-Driven Development

A workflow for developing software with Claude using structured specifications, living architecture docs, and automated validation. Each skill is a step in a repeatable loop: understand → plan → execute → review → learn.

---

## How it works

```
Project setup (once)
  /arch-create          ← document the codebase
  /scaffold             ← build the validation pipeline

Per feature (repeating loop)
  /task-brainstorm      ← explore the idea, produce a PRD
  /task-planner         ← turn PRD into an executable task file
  /task-runner          ← implement item by item, validate continuously
  /task-review          ← parallel code review (arch + style + correctness)
  /task-cleanup         ← archive task files, commit

After any session
  /introspect           ← capture feedback, improve guidelines & skills
  /arch-update          ← sync ARCHITECTURE.md after significant changes
```

---

## Directory layout: `.claude/` vs `.context/`

Per project, the workflow uses two roots with a clear separation:

- **`.claude/`** — tooling configuration that is hand-edited or scaffolded once. Persistent rules, scripts, skills, and learned preferences.
- **`.context/`** — generated workflow artifacts. Architecture docs, PRDs, task files, review tasks, and archived logs. Treat as machine-managed.

This split keeps regenerated artifacts out of the same directory as your durable configuration.

| Path | Owner | Purpose |
|---|---|---|
| `.claude/CLAUDE.md` | hand-edited / scaffolded | Hard rules and guardrails |
| `.claude/INTROSPECTION.md` | introspect skill | Learned patterns and preferences |
| `.claude/scripts/validate.sh` | scaffold skill | Project validation pipeline |
| `.claude/skills/validate-changes/SKILL.md` | scaffold skill | Thin wrapper around `validate.sh` |
| `.context/docs/ARCHITECTURE.md` | arch-create / arch-update | Machine-readable project map |
| `.context/tasks/<name>.prd.md` | task-brainstorm | Product requirements doc |
| `.context/tasks/<name>.tasks.md` | task-planner | Structured task file |
| `.context/tasks/<name>.architecture.md` | task-planner | Companion architecture scope for the task |
| `.context/tasks/<name>.review-tasks.md` | task-review | Consolidated review fixes |
| `.context/logs/<date>-<name>/` | task-cleanup | Archived task files + git hashes |

---

## Project setup (run once)

### `/arch-create`
Analyzes the codebase and generates `.context/docs/ARCHITECTURE.md` — a machine-readable reference document covering the tech stack, modules, data flow, storage, and API surface. Every planning and execution skill reads this file to avoid re-exploring the codebase from scratch each session.

Run this before anything else. If `ARCHITECTURE.md` is missing, `task-brainstorm`, `task-planner`, and `task-runner` will warn you. If it already exists, `/arch-create` offers to redirect you to `/arch-update` instead of overwriting.

### `/scaffold`
Interactively builds a project-specific validation pipeline:
- Detects your tooling (linters, type checkers, test runners, formatters)
- Asks you to confirm or adjust each detected command
- Writes `.claude/scripts/validate.sh` — runnable from terminal, CI, or any agent
- Creates a `validate-changes` skill wrapper and scaffolds doc stubs

Run this once per project, before executing tasks. The validator script is what `task-runner` calls at every validation gate.

---

## Per-feature loop

### `/task-brainstorm`
An interactive conversation that draws out your thinking about a new feature or change. Reads `ARCHITECTURE.md` first so its questions are specific to your codebase.

Covers: motivation, scope, affected modules, constraints, non-goals, risks, and alternatives. Produces a Product Requirements Document (PRD) saved to `.context/tasks/<name>.prd.md`.

Use this when the idea is vague or early-stage. If the feature is already well-defined, you can write the PRD yourself and skip to `/task-planner`.

### `/task-planner`
Reads the PRD and produces a structured, executable task file. Requires a PRD — it will stop and redirect you to `/task-brainstorm` if none exists.

Analyzes the codebase deeply, then breaks work into **phases** and **TODO items**:
- Each phase is a shippable increment (works end-to-end, safe to deploy)
- Each TODO item is fully self-contained with file path, context, and requirements
- New or structurally changed modules are flagged as **TDD modules** — the runner will scaffold stubs and failing tests before implementation

Output: `.context/tasks/<name>.tasks.md` + `.context/tasks/<name>.architecture.md`

### `/task-runner [task-file-name]`
Executes the task file one item at a time. Reads `ARCHITECTURE.md`, `CLAUDE.md`, and `INTROSPECTION.md` before starting.

For each phase entry, scaffolds TDD stubs and failing behavior tests for any listed TDD modules. Then implements each TODO item, marks it complete, and runs validation at a frequency you choose (every N items, or at the end).

Validation calls the `validator` agent, which runs your `.claude/scripts/validate.sh`. Issues are fixed inline before continuing. At each checkpoint you can continue, pause to introspect, or stop.

When all items are done, offers to run `/task-cleanup`.

### `/task-review`
Runs three review agents **in parallel**: architecture consistency, code style, and code correctness. Each writes intermediate findings, which are then consolidated into a single prioritized fix file:

- `.context/tasks/<task-name>.review-tasks.md`

The intermediate per-reviewer files are deleted after consolidation. Items are sorted CRITICAL → MAJOR → MINOR → NITPICK.

Run after implementing a feature and before opening a PR. Then use `/task-runner <task-name>.review-tasks` to execute the fixes.

### `/task-cleanup [task-file-name]`
Archives a completed task:
- Moves `.tasks.md`, `.prd.md`, and `.architecture.md` to `.context/logs/<date>-<task-name>/`
- Captures git commit hashes before and after
- Writes a `metadata.md` log entry
- Optionally invokes `/arch-update`
- Prompts for a commit message and a final commit confirmation before staging and committing

Guards against committing directly to protected branches (`master`, `main`, `dev`, `develop`, `trunk`) — offers to create a `task/<name>` branch instead.

---

## Continuous improvement

### `/introspect`
A session debrief that captures feedback, writes it to `INTROSPECTION.md`, then distills recurring patterns into `CLAUDE.md` rules.

**Phase 1–3 — Capture:** Asks what went well or poorly, classifies each item (coding pattern, workflow preference, skill improvement), proposes a target file and scope (local vs global), and appends it to `INTROSPECTION.md` without deduplication. Multiple notes on the same topic are intentional — they become signal for the next phase.

**Phase 4 — Pattern review:** After writing, re-reads `INTROSPECTION.md` and surfaces clusters of 2+ entries sharing the same theme. For each cluster it shows you the entries and their count, then asks:
- **Migrate to CLAUDE.md** — distills the cluster into a single concise agentic rule and saves it to the project or global `CLAUDE.md`, then removes the source entries
- **Leave as-is** — keeps all entries in `INTROSPECTION.md`
- **Delete these** — removes the entries without migrating

This creates a self-improving feedback loop: raw session notes accumulate in `INTROSPECTION.md`, and once a pattern recurs enough to be worth a hard rule, it gets promoted to `CLAUDE.md` where it applies to every future session.

Targets:
- `INTROSPECTION.md` — raw session notes, patterns, pitfalls (append-only until promoted)
- `CLAUDE.md` — distilled hard rules for agentic systems
- A specific `SKILL.md` — improvements to a skill's behavior

Run at the end of any session where you noticed patterns worth capturing.

### `/arch-update`
Updates `.context/docs/ARCHITECTURE.md` after significant architectural changes (new modules, new dependencies, changed data flow, new storage). Uses `git diff` and session context to identify what changed; only edits affected sections.

Run after completing a feature that changed the project's structure.

---

## Tips

- **Don't skip `ARCHITECTURE.md`.** Without it, planning skills fall back to generic analysis and lose precision.
- **PRDs are required by `task-planner`.** Write one yourself or generate it with `/task-brainstorm`.
- **Validation frequency matters.** Validating after every item is slower but catches regressions immediately. Validating at the end is faster but may require more backtracking.
- **`/introspect` improves the loop.** Patterns captured in `INTROSPECTION.md` and `CLAUDE.md` are loaded at the start of every task, so improvements compound over time.
- **`/task-review` before PRs.** Three parallel reviewers catch more than one sequential pass.
