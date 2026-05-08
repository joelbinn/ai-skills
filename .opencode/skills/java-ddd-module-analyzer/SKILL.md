---
name: java-ddd-module-analyzer
description: Identifies suitable modules/bounded contexts in a Java codebase based on a java-use-case-analyzer report. Applies DDD principles like Aggregate Roots, Bounded Contexts, Low Coupling, and High Cohesion.
---

# Java DDD Module Analyzer

## Overview
This skill takes a functional analysis report and synthesizes it into a modular architecture proposal using Domain-Driven Design (DDD) principles. The goal is to identify stable boundaries, minimize coupling, and maximize cohesion.

## Analysis Workflow
### 0. Get use case report
Use the skill `java-use-case-analyzer` to get a use case report. 
- **Use case report**: This report shall be appended as an appendix to the result report from this skill.

### 1. Group Use Cases by Lifecycle and Data
Analyze the identified use cases and system operations:
- **Common Data**: Which operations modify the same sets of entities?
- **Lifecycle Affinity**: Which entities are created, updated, and deleted together?
- **Business Capability**: Group operations that serve a specific high-level business goal (e.g., "Order Fulfillment", "Identity Management").

### 2. Identify Potential Bounded Contexts
Define boundaries based on the groupings:
- **Linguistic Boundaries**: Look for terms that change meaning in different contexts (e.g., "User" in Auth vs. "Customer" in Sales).
- **Independence**: Can this group of operations function with minimal synchronous dependence on others?
- **Team Alignment**: Could this boundary be owned by a single small team?

### 3. Identify Aggregate Roots
Within each Bounded Context, identify the core consistency boundaries:
- **Identity**: Which entity acts as the entry point for a cluster of related objects?
- **Invariants**: What rules must be enforced across multiple entities simultaneously? (The Aggregate Root is responsible for this).
- **Transaction Boundaries**: Ensure that one transaction only modifies one Aggregate instance.

### 4. Evaluate Coupling and Cohesion
Refine the boundaries:
- **High Cohesion**: Ensure that everything inside a module is highly related and changes for the same reasons.
- **Low Coupling**: Minimize dependencies between modules. Prefer asynchronous communication for cross-context side effects.
- **Independent Relocatability**: Design each module to be sufficiently decoupled so it can be easily moved to its own **Maven module** or **Microservice** without cascading changes. This requires clear public APIs and minimal shared state.

### 5. Define Integration Mechanisms
Propose how modules interact:
- **Domain Events**: Internal to a context or shared via an event bus for side effects in other contexts (e.g., `OrderPlaced` -> `InventoryReservation`).
- **Integration Events**: Explicitly designed for cross-module communication, often containing less internal detail than domain events.
- **Shared Kernel**: Use only if absolutely necessary for shared models that are too costly to duplicate.
- **Anti-Corruption Layer (ACL)**: Use when a module must interact with a legacy or differently structured context.
- **REST/gRPC APIs**: For synchronous queries or commands when immediate response is required.

## Proposed Output Structure

### 1. Bounded Context Map
A high-level overview (including a PlantUML diagram) of the identified modules and their relationships.
- **Context Name**
- **Description & Responsibility**
- **Key Aggregate Roots**

### 2. Detailed Module Breakdown
For each identified module:
- **Aggregates & Entities**: List the primary domain objects.
- **Services**: List the domain and application services.
- **Domains**: List the domain events.
- **Use Cases**: List which system operations belong here.
- **Internal Logic**: Briefly describe the core business rules managed.

### 3. Integration Architecture
A plan for how modules communicate:
- **Interaction Diagram**: (PlantUML) Showing flow between contexts.
- **Event Catalog**: List of key events (e.g., `OrderPlaced`, `CustomerRegistered`).
- **API Surface**: Key public interfaces for each module.

### 4. Justification
Explain *why* these boundaries were chosen:
- How does this design improve cohesion?
- How was coupling reduced compared to the current state?
- How does this support business scalability?

### 5. Appendices
- **Use Case Report**: the result from the skill `java-use-case-analyzer`.

## Guidelines
- **Aggregate Rule**: References between aggregates should be by ID only, not object references.
- **Event-First**: When in doubt, prefer events over synchronous calls to maintain temporal decoupling.
- **Context is King**: Avoid "Global" or "Common" modules; duplicate small pieces of data if it preserves context independence.
- **Modular Autonomy**: Each module should own its own database schema (logical or physical) and have no direct binary dependencies on the internal logic of other modules. External communication must happen through well-defined API contracts or events.
