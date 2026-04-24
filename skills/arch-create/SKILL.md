---
name: arch-create
description: Analyze a project and generate a .claude/docs/ARCHITECTURE.md file documenting its tech stack, structure, data flow, storage, and API surface.
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Bash, Write, AskUserQuestion
model: opus
---

Analyze the current project and produce `.claude/docs/ARCHITECTURE.md` — a reference document that enables agentic tools to understand and work with the project without needing to explore the codebase from scratch. Optimize all content for machine consumption: be precise, structured, and information-dense.

## Phase 0: Clarifying questions

Before analyzing, ask the following questions using `AskUserQuestion`. Ask all three in a single call.

1. **Source entry point** — What is the entry point to the source code? (e.g., `src/`, `app/`, `lib/`)
2. **Legacy exclusions** — Are there parts of the source code that are legacy and should be excluded from the architecture document? If yes, which paths?
3. **Existing documentation** — Is there existing documentation that should be read for context? If yes, what is the path?

Use the answers to scope all subsequent phases:
- Start analysis from the provided entry point.
- Exclude any legacy paths from all phases.
- If documentation was provided, read it first and use it to inform your analysis.

## Analysis phases

### Phase 1: Detect tech stack and dependencies

Scan for manifest and config files (package.json, Cargo.toml, pyproject.toml, go.mod, Gemfile, composer.json, Makefile, etc.). From these, identify:

- Languages
- Frameworks and major libraries
- Package manager
- Build tools, bundlers, dev servers
- Test runners, linters, formatters
- 3rd party dependencies: list each with its purpose/role in the project (do NOT include version numbers)

### Phase 2: Map project structure and modules

Explore the directory tree and identify major modules. For each module:

- Its directory path and key files
- Its purpose: what problem it solves or what domain it owns
- How it works: its internal structure, key abstractions, and the pattern it follows (e.g., "controller/service/repository", "composable wrapping an API client", "event-driven pipeline")
- What it depends on and what depends on it

Do not list every file. Focus on directories and key files that define module boundaries. Go deep enough that an agent can understand what a module does and how to work within it without reading every file.

### Phase 3: Trace data flow

Read entry points, route definitions, API handlers, state management, and service layers. Document data flow as written explanations organized by context (e.g., "User authentication", "Order processing", "File upload"). For each context, explain:

- How data enters (HTTP requests, CLI input, event listeners, message queues, etc.)
- How it moves between layers and which modules are involved
- Key transformations or processing steps
- How data exits (responses, writes, emitted events, etc.)

Do NOT generate ASCII diagrams or visual graphs. Use structured prose and bulleted sequences.

### Phase 4: Inventory data storage

Identify all data persistence mechanisms:

- Databases (type, ORM/driver, what data they store)
- Caches (Redis, in-memory, etc. and their purpose)
- File storage (uploads, generated files, logs)
- External services used for storage (S3, cloud storage, etc.)
- Client-side storage (localStorage, IndexedDB, cookies — if applicable)

### Phase 5: Document entry points and API surface

Identify:

- Application entry points (main files, route registrations, CLI commands)
- External API endpoints or interfaces the project exposes
- External APIs or services the project consumes
- Key internal boundaries between modules

## Output

Write the file `.claude/docs/ARCHITECTURE.md` using this structure:

```markdown
# Architecture

## Tech stack
[Bulleted list: language, framework, package manager, build tools, test/lint tooling]

## Dependencies
[Bulleted list of 3rd party dependencies grouped by concern (e.g., UI, state management, HTTP, validation, testing). Each entry: dependency name — what it does in this project. No version numbers.]

## Modules
[For each major module:]

### [Module name] (`path/`)
- **Purpose**: [what this module owns]
- **How it works**: [internal pattern, key abstractions, processing approach]
- **Key files**: [files that define the module's boundaries and behavior]
- **Depends on**: [other modules it imports from]
- **Depended on by**: [modules that import from it]

## Data flow
[Organized by context/use case. For each context:]

### [Context name] (e.g., "User authentication")
[Bulleted sequence explaining: entry point → layers involved → transformations → output. Reference module names from the Modules section.]

## Data storage
[Bulleted list of each storage mechanism: type, technology, what data it holds, and its role in the system]

## Entry points and API surface

### Exposed
[Entry points and APIs this project exposes — routes, CLI commands, exported modules]

### Consumed
[External APIs and services this project depends on]
```

## Rules

- This document is for agentic tools, not humans. Optimize for information density and precise references (paths, module names) over readability.
- Be information-dense. Every line should add a fact an agent needs to navigate or modify the codebase.
- Do not document individual functions. Document modules: their purpose, how they work, and their relationships.
- Do not include dependency version numbers.
- Do not generate ASCII diagrams or visual graphs. Use structured prose and bulleted sequences.
- Only document what you can confirm from the code. Do not speculate.
- If a section is not applicable (e.g., no data storage in a pure library), omit it.
