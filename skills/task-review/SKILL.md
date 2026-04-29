---
name: task-review
description: Run all code review agents in parallel (architecture, style, correctness), consolidate findings into a single fix-tasks file, and remove the individual review files.
user-invocable: true
disable-model-invocation: true
allowed-tools: Agent, Bash, AskUserQuestion, Read, Write
model: sonnet
---

# Code Review

Launch all review agents in parallel. Each agent independently analyzes the current git changes from its own perspective and writes its findings to a temporary file. All findings are then consolidated into a single prioritized task file.

## Step 1: Confirm scope

Run `git diff --name-only HEAD` and `git status --porcelain`. If there are no changed files, tell the user there is nothing to review and stop.

Derive a `<task-name>` for the output file: use the current git branch name (strip the `task/` prefix if present), falling back to the kebab-cased basename of the first changed file if the branch name is generic (`main`, `master`, `dev`, `develop`, `trunk`).

## Step 2: Launch all agents in parallel

Use the Agent tool to launch all three agents **simultaneously in a single message** (one Agent tool call per agent, all in the same response):

| Agent | subagent_type | Output file |
|---|---|---|
| architecture-reviewer | `architecture-reviewer` | `.context/tasks/architecture-fix-tasks.md` |
| code-style-reviewer | `code-style-reviewer` | `.context/tasks/code-style-fix-tasks.md` |
| code-correctness-reviewer | `code-correctness-reviewer` | `.context/tasks/code-correctness-fix-tasks.md` |

Each agent is self-contained — pass no prompt, they load their own context.

## Step 3: Consolidate into a single task file

Once all agents complete:

1. Read each of the three output files.
2. Collect all `- [ ]` task items across all files. Ignore files that contain `## No Issues Found`.
3. Sort all items by severity: `CRITICAL` → `MAJOR` → `MINOR` → `NITPICK`. Within the same severity, group by reviewer (architecture, then correctness, then style).
4. Write `.context/tasks/<task-name>.review-tasks.md` using this format:

```markdown
# Review Fix Tasks: <task-name>

*<YYYY-MM-DD> | <branch>*

## Context

Combined output from 3 parallel review agents:
- Architecture: N issues
- Correctness: N issues
- Style: N issues

## Tasks

- [ ] [CRITICAL] ...
  - **Reviewer**: architecture | correctness | style
  - **File**: `path/to/file.ts:line`
  - **Problem**: ...
  - **Fix**: ...

- [ ] [MAJOR] ...
  ...
```

5. Delete the three individual review files:
   ```
   rm .context/tasks/architecture-fix-tasks.md
   rm .context/tasks/code-style-fix-tasks.md
   rm .context/tasks/code-correctness-fix-tasks.md
   ```

## Step 4: Report

```
## Review Complete

3 agents ran in parallel. Findings consolidated into `.context/tasks/<task-name>.review-tasks.md`:

- Architecture: N issues
- Correctness: N issues
- Style: N issues
- Total: N issues (CRITICAL: n, MAJOR: n, MINOR: n, NITPICK: n)

Run `/task-runner <task-name>.review-tasks` to execute all fixes.
```

If all three agents found no issues, report that and do not create the consolidated file.
