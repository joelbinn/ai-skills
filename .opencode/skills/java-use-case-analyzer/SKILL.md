---
name: java-use-case-analyzer
description: Analyzes a Java codebase to identify and document all use cases and system operations. Identifies entry points (REST, Scheduled, etc.) and uses java-call-tree-analyzer to map the full functional flow.
---

# Java Use Case Analyzer

## Overview
This skill is used to perform a comprehensive functional analysis of a Java-based application. It identifies all "call roots" (external entry points) and traces their execution paths to document the system's behavior as a set of use cases and system operations.

# Rules
- Just use vanilla PlantUML. **Never** use any extensions for C4 diagrams.

## Analysis Workflow

### 1. Call Root Identification
Scan the codebase to identify all entry points where external actors or systems trigger logic:
- **REST Endpoints:** Search for `@RestController` or `@Controller` with `@RequestMapping`, `@GetMapping`, `@PostMapping`, etc.
- **Scheduled Jobs:** Search for `@Scheduled` methods.
- **Message Listeners:** Search for `@RabbitListener`, `@KafkaListener`, `@JmsListener`, etc.
- **Event Listeners:** Search for `@EventListener` or `ApplicationListener` implementations.
- **CLI / Main:** Search for `public static void main` or `CommandLineRunner`/`ApplicationRunner` implementations.

### 2. Deep Functional Analysis & Terminology Discovery
For each identified call root:
1.  **Echo**: Echo that the specific entry point has been identified in the terminal.
2.  **Invoke a subagent for `java-call-tree-analyzer`**:
    - If the tool `task` is available, use the `task` tool with `subagent_type: general` to launch a subagent; otherwise try to spawn a subagent if possible or just do the following in the main agent.
    - Instruct the subagent to load and follow the `java-call-tree-analyzer` skill to perform a deep recursive analysis of the method.
    - Ensure the subagent returns the result in the format specified by the `java-call-tree-analyzer` report structure.
    - Save the result in an appendix of the result report of this skill.
    - Add the result to section `D. Call Tree Analysis` of the final report.
3.  **Extract Use Case Details**: From the call tree analysis, identify:
    - The primary actor (User via API, System via Timer, External System via Message).
    - The qualified name of the method that is the entry point.
    - The core business intent.
    - Key data transformations and validations.
    - Consistency boundaries (Aggregates) affected.
    - External systems contacted.
4.  **Identify Domain Terminology**: During the analysis, capture key domain concepts:
    - **Entities & Value Objects**: Look for naming patterns that represent core business concepts.
    - **Domain Events**: Identify significant state changes or signals.
    - **Ubiquitous Language**: Note terms that appear consistently in class names, method names, and comments.

### 3. Documentation & Synthesis
Synthesize the findings into a structured report.

## Report Structure

### 1. System Use Cases / Operations / Queries
A catalog of all identified operations.
- **Operation Name:** [e.g., Place Order, Synchronize Inventory]
- **Trigger:** [e.g., POST /api/v1/orders, Cron expression]
- **Type:** Query (non-mutating) or Command (mutating)
- **Root method:** The qualified name of the method that triggered the operation, e.g. `org.springframework.samples.petclinic.owner.OwnerController.findOwner`.
- **Description:** A concise description of the functional flow and business goal.
- **Impact:** Entities/Aggregates modified or queried.
- **Included classes:** List of classes involved in the implementation of the operation.

### 2. C4 Level 1 - System Context Diagram
Provide a PlantUML diagram representing the System Context.
- Include the **System Under Study**.
- Include **People** (Actors) interacting with it.
- Include **External Systems** identified during analysis.

### 3. Appendix: Entry Point Details

#### A. REST API Description
Detailed list of all REST endpoints:
- **Method & Path**
- **Description**
- **Input/Output (brief)**

#### B. Other Call Roots
List of non-REST entry points:
- **Scheduled Jobs:** Method name, schedule, and purpose.
- **Message Listeners:** Source (Queue/Topic) and purpose.
- **Other:** Event listeners, CLI commands, etc.

#### C. Domain Terminology
A glossary of key concepts identified during the analysis:
- **Term:** [e.g., Aggregate Root / Value Object name]
- **Context:** [Where it is used]
- **Description:** Business meaning and role in the system.

#### D. Call Tree Analysis
The call tree analysis result (the exact report) from the skill `java-call-tree-analyzer` of each 
identified entry point. It *MUST* include
- Affected Entities
- External System Integrations
- Call Tree
- Complete Functional Description
- Test assertions

## Guidelines
- Focus on the *Domain* and *Application* layers when describing use cases.
- Use the terminology found in the codebase (Domain-Driven Design).
- Ensure the C4 diagram reflects all external integrations found in the call tree analysis.
