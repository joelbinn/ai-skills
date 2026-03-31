---
name: codebase-analyzer
description: >
  Performs deep analysis of a software project's codebase and produces a
  structured description of its functional characteristics. Use this skill
  when asked to "analyze the codebase", "describe what this application does",
  "map out the functionality", or "produce a functional overview".
---

# Codebase Analyzer

This skill guides the agent through a systematic, multi-pass analysis of a
software project's codebase. The goal is to produce a clear, accurate
description of the application's **functional characteristics** — what it does,
for whom, and how its capabilities are organized — without requiring prior
knowledge of the project.

---

## Workflow

### Phase 1 — Orient

Before reading any source code, establish the project's context.

1. Locate and read top-level orientation files in this order:
   - `README.md` / `README` (any extension)
   - `GEMINI.md` / `AGENTS.md` / `CLAUDE.md`
   - `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`,
     `build.gradle`, or equivalent manifest
   - `docs/` or `documentation/` directory index (if present)
   - `*.md` files

2. From these files, extract:
   - **Project name and stated purpose**
   - **Target users or personas**
   - **Technology stack** (languages, frameworks, runtimes)
   - **Entry points** (main scripts, CLI commands, web server start-up, etc.)

3. List the top-level directory structure to understand the project layout.
   Identify which directories likely contain source code, tests, configuration,
   and build artefacts.

---

### Phase 2 — Map the Structure

Identify the major structural units of the codebase.

1. Walk the source directories (not test or build directories).
2. Group files into **modules, packages, layers, or services** based on
   directory names and import patterns.
3. For each identified unit, note:
   - Its apparent responsibility (UI, API, data access, business logic, etc.)
   - Its primary entry file or public interface
   - Whether it appears to be a library, service, CLI tool, or UI component

4. If a dependency graph is inferable (e.g., from `import` statements,
   `require()`, or build config), sketch the high-level call flow between units.

---

### Phase 3 — Identify Core Features

For each structural unit identified in Phase 2, read the key source files and
extract the concrete capabilities they implement.

Focus on:
- **Public functions, classes, routes, commands, and event handlers** — these
  represent user-facing or integration-facing behaviour.
- **Data models and schemas** — these reveal what domain concepts the
  application manages.
- **External integrations** — APIs called, databases queried, message queues
  consumed, file formats processed.
- **Configuration surface** — environment variables, feature flags, or settings
  that change runtime behaviour.

For each feature found, record:
- A concise name (one to five words)
- A one-sentence description of what it enables
- Which structural unit owns it
- Any notable constraints, preconditions, or dependencies

---

### Phase 4 — Synthesize

Combine the raw findings into a coherent narrative.

Produce the following sections in your final output:

#### 4.1 Application Summary
Two to four sentences describing:
- What the application is
- Who it is for
- What problem it solves or value it provides

#### 4.2 Technology Overview
A brief table or list of:
- Runtime / language version(s)
- Core frameworks and libraries
- Infrastructure dependencies (databases, queues, storage, etc.)

#### 4.3 Functional Capabilities
A structured list of the application's features grouped by domain or layer.
For each feature:
- **Name** — short label
- **Description** — one to two sentences
- **Owner** — which module/service/layer provides it
- **Key dependencies** — notable external services or libraries it relies on

#### 4.4 Data Domain
A description of the core domain entities managed by the application:
- Entity names and their roles
- Relationships between entities (if discernible)
- Persistence mechanisms used (SQL, NoSQL, file system, in-memory, etc.)

#### 4.5 Integration Points
A list of all external systems the application communicates with:
- Name or type of system
- Direction (inbound / outbound / bidirectional)
- Protocol or mechanism (REST, gRPC, SDK, file, webhook, etc.)
- Purpose

#### 4.6 Configuration and Deployment
Summary of how the application is configured and deployed:
- Configuration method (env vars, config files, CLI flags)
- Build and packaging approach
- Deployment targets (cloud, container, bare metal, desktop, etc.)

#### 4.7 Gaps and Uncertainties
Honest list of areas where the analysis is incomplete or uncertain, with
reasons (e.g., "No database schema file found — inferred from ORM models only").

---

## Output Format

- Write the output in the same language the user used when invoking the skill,
  unless they specify otherwise.
- Use Markdown with clear headings matching the sections in Phase 4.
- Be specific and concrete — avoid vague generalities like "handles business
  logic". Name the actual domain concepts and operations.
- Do not fabricate capabilities. If a feature cannot be confirmed by reading
  the code, mark it as inferred and state the basis for the inference.
- Keep the summary section short enough to be useful as a quick reference
  (aim for under 200 words). The full capability list may be as long as needed.

---

## Constraints

- Do **not** modify any files during analysis. This skill is read-only.
- Do **not** run tests, builds, or install dependencies unless the user
  explicitly requests it.
- If the codebase is very large (thousands of files), apply sampling: read all
  entry points and public interfaces, then sample representative files from
  each structural unit rather than reading every file.
- Prioritize breadth over depth in Phase 2–3; a shallow understanding of all
  parts is more useful than an exhaustive understanding of one part.
