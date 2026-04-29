---
name: task-cleanup
description: Archive a completed task — move task files to logs, capture git hashes, and create a commit.
argument-hint: task-file-name (optional)
user-invocable: true
disable-model-invocation: false
allowed-tools: Read, Bash, Write, Edit, Glob, AskUserQuestion, Skill
model: sonnet
---

Archive a completed task by migrating its files to a log directory, recording git state, and committing the work.

## Phase 0: Find the task file

- If `$ARGUMENTS` is provided: look for `.context/tasks/$ARGUMENTS`, then `.context/tasks/$ARGUMENTS.tasks.md`, then `.context/tasks/$ARGUMENTS.md`.
- Otherwise: find the first `.md` in `.context/tasks/` that has all items checked (`- [x]`). Ignore `.architecture.md` and `.prd.md` files — those are companion documents, not task files.
- If no task file is found, inform the user and stop.
- Derive `<task-name>` from the filename: strip `.tasks.md` if present, otherwise strip `.md`.
- Run `date -u +%Y-%m-%d` and `date -u +%Y-%m-%dT%H:%M:%SZ` to capture the current date and full datetime in UTC.
- Derive `<log-dir-name>` as `<date>-<task-name>` (e.g., `2026-03-24-my-task`).

### Unchecked items guard

If the task file contains any unchecked `- [ ]` items, use `AskUserQuestion` to warn the user:

> **Warning: task is not fully complete**
> `<task-name>` still has unchecked items. Archiving now will capture an incomplete task.
>
> Options:
> - Proceed anyway
> - Stop — I'll finish the remaining items first

If the user chooses to stop, exit.

## Phase 0.5: Protected branch guard

1. Run `git rev-parse --abbrev-ref HEAD` to get the current branch name.
2. If the branch is `master`, `trunk`, `main`, `dev`, or `develop`, use `AskUserQuestion`:

> **You are on a protected branch: `<current-branch>`**
> Committing task work directly to `<current-branch>` is not recommended.
>
> Options:
> - **A. Commit as-is** — proceed on `<current-branch>`
> - **B. Create a gitflow branch** — checkout a new branch `task/<task-name>` and commit there
> - **C. Abort** — stop without making any changes

- If **A**: continue as normal.
- If **B**: run `git checkout -b task/<task-name>` then continue.
- If **C**: exit.

## Phase 1: Resolve log directory

Determine the target log directory: `.context/logs/<log-dir-name>/`.

If that directory already exists, use `AskUserQuestion`:

> **Log directory already exists**
> `.context/logs/<log-dir-name>/` already exists.
>
> Options:
> - Overwrite — replace existing files
> - Append suffix — use `.context/logs/<log-dir-name>-2/` (increment until free)
> - Abort

If the user chooses to abort, exit. If suffix, find the next available integer suffix.

## Phase 2: Capture pre-commit state

1. Run `git rev-parse HEAD` to get the current commit hash.
2. Create the log directory.
3. Write `.context/logs/<log-dir-name>/metadata.md`:

```markdown
# Task Log: <task-name>

## Archived At
<full datetime (yyyy-mm-ddTHH:MM:SSZ, UTC)>

## Commit Before
<hash>

## Commit After
_(pending)_
```

## Phase 3: Move task files

Strip the `<task-name>` prefix as you move each file into the log directory so the archived files are named by their role, not the task:

- Move the task file → `.context/logs/<log-dir-name>/tasks.md`
  - Source is `.context/tasks/<task-name>.tasks.md` if it exists, otherwise `.context/tasks/<task-name>.md`.
- If `.context/tasks/<task-name>.architecture.md` exists, move it → `.context/logs/<log-dir-name>/architecture.md`
- If `.context/tasks/<task-name>.prd.md` exists, move it → `.context/logs/<log-dir-name>/prd.md`

Use `mv` via Bash for each.

## Phase 3.5: Architecture update

Use `AskUserQuestion` to ask:

> **Update architecture docs?**
> Would you like to run `/arch-update` to sync `ARCHITECTURE.md` with the changes made in this task?
>
> Options:
> - **Yes** — run arch-update now (recommended if new modules, dependencies, or data flows were introduced)
> - **No** — skip

If the user chooses **Yes**: invoke the `arch-update` skill via the Skill tool. Wait for it to complete before continuing.

## Phase 4: Generate and confirm commit message

1. Read the archived task file.
2. If a `## Context` section exists, use it as the primary source to generate the commit message. Otherwise, infer intent from the completed TODO items.
3. Choose an appropriate commit style (conventional commit prefix, plain summary, etc.) based on the task type and context.
4. Use `AskUserQuestion`:

> **Commit message**
> Here's the suggested commit message:
> ```
> <generated message>
> ```
>
> Options:
> - Accept as-is
> - Provide a different message

Use the accepted or user-provided message.

## Phase 5: Commit and finalize

1. Run `git status --porcelain` and show the user the list of files that will be staged.
2. Use `AskUserQuestion`:

> **Ready to commit**
> The files above will be staged with `git add -A` and committed with the message confirmed in Phase 4.
>
> Options:
> - **Commit now** — proceed with `git add -A` and commit
> - **Stage manually then commit** — pause so the user can stage selectively, then resume
> - **Abort** — leave the archived files in place but make no commit

- If **Commit now**: run `git add -A`, then create the commit using the confirmed message.
- If **Stage manually then commit**: instruct the user to stage what they want, then ask via `AskUserQuestion` whether to proceed; on proceed, run `git commit` (no `add -A`) using the confirmed message.
- If **Abort**: skip steps 3–4 and report that the archive is in place but no commit was made.

3. Run `git rev-parse HEAD` to get the new commit hash.
4. Update `.context/logs/<log-dir-name>/metadata.md` — replace `_(pending)_` with the new hash.
