# Task File Format

This is the canonical format for task files.
Task files live in `.claude/tasks/<descriptive-kebab-name>.tasks.md`.

## Task File Template

```markdown
# Task: <title>

## Metadata
- **Type**: feature | update | refactor | tech-debt
- **Created**: <date>
- **Architecture**: ./<task-name>.architecture.md

## Context
<Brief description of the task, goals, and constraints.>
<Relevant modules from ARCHITECTURE.md and why they are involved.>
<Key patterns or conventions to follow.>

## Sub-tasks

### Phase 1: <phase name>
**Deliverable**: <one sentence — what end-user value this phase ships>
**TDD modules** (new files or files whose exported interface changes):
- `path/to/file.ts` — <key behaviors / interface intent to test>

- [ ] **<item title>**
  - file: `path/to/file:line`
  - context: <why this change is needed — what problem it solves or what it enables for subsequent tasks>
  - instructions: <what to do, expressed as requirements and constraints, NOT implementation details — no variable names, no function signatures, no CSS values, no code patterns>
  - depends-on: none

- [ ] **<item title>**
  - file: `path/to/file`
  - context: <why this change is needed>
  - instructions: <what to do, as requirements and constraints>
  - depends-on: <title of dependency item>

### Phase 2: <phase name>
**Deliverable**: <one sentence>
**TDD modules**: none
- [ ] ...
```

## Companion Architecture Template

Write to `.claude/tasks/<task-name>.architecture.md` when applicable:

```markdown
# Task Architecture: <title>

## Scope
<Which modules from ARCHITECTURE.md are involved and why>

## Affected Files
<List of files that will be created, modified, or deleted>

## Key Patterns
<Relevant patterns from the codebase that implementations should follow>

## Data Flow
<How data flows through the affected area, specific to this task>

## Constraints
<Things that must not change, edge cases, backward compatibility needs>
```
