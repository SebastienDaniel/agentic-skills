---
name: arch-update
description: Update .claude/docs/ARCHITECTURE.md to reflect changes made in the current session. Uses session context and git diff to identify what changed.
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Edit
model: opus
---

Update the existing `.claude/docs/ARCHITECTURE.md` to reflect significant architectural changes made in the current session.

Run fully autonomously. Do not ask the user any questions.

## Step 1: Understand what changed

1. Read the existing `.claude/docs/ARCHITECTURE.md` to understand the current documented state.
2. Run `git diff HEAD` to see all uncommitted changes. If there are no uncommitted changes, run `git diff HEAD~1` to see the most recent commit's changes.
3. From both the session context and the diff, identify changes that are architecturally significant:
   - New modules, directories, or major files added or removed
   - New or removed dependencies (frameworks, libraries, services)
   - Changes to data flow (new API endpoints, changed routing, new data pipelines)
   - Changes to data storage (new databases, caches, storage mechanisms)
   - New or removed entry points or external API surface
   - Changes to project structure or module boundaries

Ignore changes that are not architecturally significant (bug fixes, style changes, minor refactors within existing modules, test additions for existing features).

## Step 2: Update the document

For each architecturally significant change:

1. Identify which section(s) of ARCHITECTURE.md are affected.
2. Use the Edit tool to update only those sections. Preserve all unaffected content exactly as-is.
3. Match the existing style and level of detail.

If the changes introduce something entirely new that doesn't fit any existing section, add it under the appropriate heading following the established document structure.

## Step 3: Report

After editing, output a brief summary of what was updated and why (2-5 bullet points).

## Rules

- If `.claude/docs/ARCHITECTURE.md` does not exist, inform the user and suggest running `/arch-create` first. Do not create the file.
- Only update sections affected by the changes. Do not rewrite or reorganize unrelated sections.
- Do not add implementation details. Stay at the same module/layer level as the existing document.
- If no changes are architecturally significant, say so and make no edits.
- Preserve any manual edits the user may have made to the document.
