---
name: java-test-suite-generator
description: Generates a comprehensive test suite for a Spring Boot application by analyzing use cases and defining test cases with clear acceptance criteria.
---

# Java Test Suite Generator

## Overview
This skill automates the creation of a test suite for Spring Boot applications. It leverages the `java-use-case-analyzer` to discover the functional surface of the application and then designs detailed test cases for each identified operation.

## Workflow

### 1. Functional Analysis
The first step is to understand what needs to be tested.
- **Invoke `java-use-case-analyzer`**: Use this skill to identify all System Use Cases, Operations, and Queries.
- **Review Results**: Focus on the "System Use Cases / Operations / Queries" section of the generated report.

### 2. Test Case Design
For each identified operation (Command or Query), generate a set of test cases following the Given-When-Then structure.

#### Test Categories to Cover:
- **Happy Path**: The most common successful execution of the use case.
- **Edge Cases**: Boundary values for inputs, empty collections, etc.
- **Negative/Error Paths**: 
    - Validation failures (e.g., 400 Bad Request).
    - Resource not found (e.g., 404 Not Found).
    - Business rule violations (e.g., 422 Unprocessable Entity).
    - Security/Permission failures (e.g., 401/403).
- **Side Effects**: Verify database updates, events published, or external system calls.

### 3. Acceptance Criteria (AC)
Each test case must include clear Acceptance Criteria:
- **State Changes**: What should change in the database/system state?
- **Response**: What is the expected return value or HTTP status code?
- **Logs/Events**: Are there specific events that must be emitted?
- **External Calls**: What interactions with mockable dependencies are expected?

## Report Structure

For each use case, provide the following:

### [Use Case Name]
**Root Method:** `fully.qualified.ClassName.methodName`

| ID | Scenario | Given (Preconditions) | When (Action) | Then (Acceptance Criteria) |
|---|---|---|---|---|
| TC-1 | Happy Path | ... | ... | ... |
| TC-2 | Invalid Input | ... | ... | ... |
| ... | ... | ... | ... | ... |

## Implementation Guidance
The test suite must always be implemented as **Integration Tests** using `@SpringBootTest`. This ensures that the full Spring ApplicationContext is started, and all beans are correctly initialized and wired.

### Implementation Rules:
- **Test Framework**: Always use `@SpringBootTest` to ensure complete bean initialization and setup.
- **Entry Points**: All tests must be executed by calling the identified entry point methods (e.g., using `MockMvc` or direct calls to `@RestController` methods in a Spring context).
- **Repositories**: All repositories must be mocked using Mockito (`@MockitoBean` or `@MockBean`). Verify repository interactions (save, find, delete) to ensure business logic is correctly persisted.
- **External Services**: Any outgoing REST calls or external system integrations must be mocked using Mockito (`@MockitoBean` or `@MockBean`) by mocking the service/client interfaces.
- **Isolation**: Ensure tests are isolated and do not require a running database or external environment, as all infrastructure and external dependencies are mocked.

## Guidelines
- **Terminology**: Use the ubiquitous language identified by `java-use-case-analyzer`.
- **Completeness**: Ensure all paths identified in the call tree (from the analyzer) are covered by at least one test case.
- **Immutability**: Ensure tests are independent and do not rely on state from previous tests.
