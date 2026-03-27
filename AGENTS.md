# AGENTS.md - AI Skills Repository

This repository contains the OpenSpec experimental workflow system with skills for AI-assisted development.

## Project Overview

- **Type**: Multi-agent skill system for AI-assisted development
- **OpenSpec**: Workflow system with skills (propose, apply, archive, explore) stored in `.opencode/`
- **Claude Skills**: Skills for Claude AI in `.claude/` directory
- **Gemini Skills**: Skills for Gemini AI in `.gemini/` directory
- **Config**: `openspec/config.yaml` defines the spec-driven schema

## Build/Lint/Test Commands

This is a documentation/workflow repository - no build commands apply.

For OpenSpec operations, use the skill commands:
- `/openspec-propose` - Propose a new change
- `/openspec-apply` - Implement tasks from a change
- `/openspec-archive` - Archive a completed change
- `/openspec-explore` - Explore ideas and clarify requirements

## Code Style Guidelines

### General Principles

- **Domain-Driven Design**: Follow layered architecture (Domain, Application, Infrastructure, Presentation)
- **Gang of Four Patterns**: Apply GoF design patterns where appropriate (Factory, Strategy, Observer, Decorator, Adapter)
- **Complexity Limits**:
  - Cyclomatic complexity: Target 1-10, max 15
  - Cognitive complexity: Target 1-10, max 15
  - Break complex functions into smaller, focused functions

### Naming Conventions

- **Classes**: PascalCase (e.g., `CustomerRepository`, `PaymentService`)
- **Functions/Methods**: camelCase (e.g., `calculateTotal`, `processOrder`)
- **Value Objects**: PascalCase, always immutable
- **Domain Events**: Past tense (e.g., `OrderPlaced`, `PaymentReceived`)
- **Repositories**: Singular noun per aggregate (e.g., `CustomerRepository`, not `CustomersRepository`)

### Architecture Patterns

**Entities**: Objects with unique identity that persist over time
**Value Objects**: Defined by attributes, always immutable, contain validation
**Aggregates**: Cluster of entities treated as a unit with Aggregate Root as entry point
**Repositories**: One per Aggregate Root, collection-like interface
**Factories**: Encapsulate complex object creation
**Domain Services**: Stateless operations involving multiple aggregates
**Application Services**: Thin orchestration layer, no business logic

### Code Structure

```
Domain Layer: Core business logic (no outer dependencies)
Application Layer: Use case orchestration
Infrastructure Layer: Technical implementation (DB, external services)
Presentation Layer: APIs and UI
```

Dependencies always point inward toward the domain.

### Error Handling

- Domain events for cross-aggregate communication (loose coupling)
- Repositories hide persistence details from domain layer
- Validate value objects at construction time
- Application Services manage transactions

### Formatting

- 2-space indentation
- Clear class names reflecting design patterns (e.g., `UserFactory`, `DatabaseAdapter`)
- Comments referencing pattern names when relevant
- Early returns to reduce nesting

## OpenSpec Workflow

This repo uses OpenSpec for experimental workflow:

1. **Propose** (`/openspec-propose`): Create change proposal with design, specs, tasks
2. **Explore** (`/openspec-explore`): Thinking partner for investigating problems
3. **Apply** (`/openspec-apply`): Implement tasks from a change
4. **Archive** (`/openspec-archive`): Finalize and archive completed changes

Changes stored in `openspec/changes/`, specs in `openspec/specs/`.

## File Locations

- Skills: `.opencode/skills/openspec-*/SKILL.md`
- Commands: `.opencode/command/`
- OpenSpec config: `openspec/config.yaml`
