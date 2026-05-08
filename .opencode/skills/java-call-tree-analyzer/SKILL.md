---
name: java-call-tree-analyzer
description: Deeply analyzes the call tree of a Java/Spring Boot method. Use when asked to trace method behavior recursively, identifying entity interactions (JPA) and external system calls within the local codebase.
---

# Java Call Tree Analyzer

## Overview
This skill provides a systematic workflow for performing a deep, recursive analysis of a specific Java method's call tree. It is designed to map the full functional scope of a method within a Spring Boot application, focusing on business logic, database entities, and external integrations while filtering out boilerplate and framework noise.

## Analysis Workflow

### 1. Entry Point Identification
- **Action:** Locate the target method specified by the user.
- **Goal:** Read and understand the entry point's signature, parameters, and immediate body.

### 2. Recursive Call Tracing
- **Action:** Identify all method invocations within the current method body.
- **Rules:**
  - **Local Calls:** If the method belongs to the current project (e.g., same package or subpackages), navigate to that method and repeat the analysis recursively.
  - **Framework/Third-Party Calls:** Stop the recursion if the call is to a standard library (`java.*`, `javax.*`, `jakarta.*`) or a third-party framework (e.g., `org.springframework.*`, `org.hibernate.*`, `com.fasterxml.jackson.*`).
  - **Infrastructure/Adapter Boundaries:** Identify calls to Repositories or External Clients as leaf nodes but note their intent.

### 3. Entity and External System Identification
As you traverse the tree, maintain a log of the following:

- **JPA Entities:** Look for classes annotated with `@Entity`.
  - Identify which attributes are accessed.
  - Determine the operation type: **READ** (getters, query methods) or **WRITE** (setters, constructors, save methods).
- **External Systems:** Look for indicators of external integration:
  - `@FeignClient` interfaces.
  - `RestTemplate` or `WebClient` usage.
  - Messaging clients (JMS, Kafka, RabbitMQ).
  - Explicit service calls to other bounded contexts or external APIs.

### 4. Functional Synthesis
- **Action:** Combine the gathered information from all levels of the call tree.
- **Goal:** Formulate a cohesive description of the "complete" behavior of the entry point method, including side effects and data flow.

## Report Format
The final output must be a structured report containing:

### 1. Affected Entities
List all JPA entities encountered in the call tree.
- **Entity Name:** [Name]
  - **Attributes:** [Attribute List]
  - **Access Mode:** [Read / Write / Both]

### 2. External System Integrations
List all external systems and the specific operations performed.
- **System/Client:** [e.g., Payment Gateway, Customer Service]
  - **Call Type:** [e.g., POST /v1/payments, GET /profile]
  - **Purpose:** [Brief description]

### 3. Complete Functional Description
A detailed narrative explaining what the method does from start to finish, incorporating the logic found in all sub-methods. Describe the data transformations, validations, and final outcomes.
