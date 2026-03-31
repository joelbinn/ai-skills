name: java-method-analyzer
description: Analyzes a Java method by tracing its execution flow, examining method bodies, and following all method calls within the project codebase. Use when asked to "explain what this method does", "trace method execution", "analyze method flow", or "understand method behavior".
license: MIT
compatibility: Works with Java projects
metadata:
  author: ai-skills
  version: "1.0"
  generatedBy: "1.0"
---

Java Method Analyzer

This skill guides the agent through a systematic analysis of a Java method. The analysis traces the method's execution flow, examines what it does in its body, and follows all method calls it makes. The analysis descends through the call graph as long as calls are made to classes in the project codebase. When external libraries or Java standard library classes are encountered, the analysis stops and makes reasonable assumptions about what those calls do.

---

## Workflow

### Phase 1 — Locate and Load the Method

1. **Gather input from user**:
   - Class name (fully qualified or simple)
   - Method name
   - Method signature (parameter types, if needed for disambiguation)
   - Project root or source directory (if not obvious from context)

2. **Locate the method file**:
   - Search the project source directories for the class file
   - If multiple matches exist (overloads), clarify with the user or analyze all relevant overloads
   - Read the complete class file to understand the method and its context

3. **Extract method metadata**:
   - Method signature (return type, parameters, modifiers)
   - Method body
   - Class name and package
   - Any Javadoc or comments explaining the method
   - Related fields or constants used by the method

---

### Phase 2 — Analyze Method Body

1. **Parse the method body** to identify:
   - Local variable declarations and assignments
   - Control flow structures (if/else, loops, switch statements)
   - Direct operations (arithmetic, string manipulation, object creation)
   - All method calls made within this method

2. **For each method call identified**:
   - Record the target class and method name
   - Record the parameters passed
   - Determine the call source:
     - Is it a call on `this` (same class)?
     - Is it a call on another local variable or field?
     - Is it a static method call?
     - Is it a constructor call?

3. **Classify the call source**:
   - **In-project call**: Method exists in the project codebase → Will analyze further
   - **External library call**: Method is from an external dependency → Assume behavior, do not descend
   - **Java standard library call**: Method is from `java.*` or `javax.*` packages → Assume behavior, do not descend

---

### Phase 3 — Trace Dependent Method Calls

For each method call identified in Phase 2:

1. **Check if it's an in-project call**:
   - If yes, locate the target method in the project
   - If the method is not found, record it as "unresolved" and note the likely reason
   - If the method is found, add it to the analysis queue

2. **Analyze dependent methods**:
   - For each in-project method call, repeat Phase 2 (analyze its body)
   - Build a call tree showing which methods call which
   - Continue descending until all in-project calls have been analyzed
   - Stop at external library or standard library calls (mark as "external" or "stdlib")

3. **Handle special cases**:
   - **Recursive calls**: Detect and note them; analyze only one level to avoid infinite recursion
   - **Conditional calls**: Note that certain calls happen only under certain conditions
   - **Loop calls**: Note if a method is called repeatedly in a loop
   - **Polymorphic calls**: If a method is called on an interface or abstract class, note the likely concrete implementations

---

### Phase 4 — Build Execution Flow Summary

Synthesize findings into a clear execution flow narrative.

Produce the following sections in your final output:

#### 4.1 Method Summary

A concise statement of what the method does:
- **Name**: Fully qualified method name (e.g., `com.example.UserService.findActiveUsers`)
- **Signature**: Method signature with return type and parameters
- **Purpose**: One to three sentences describing the method's role and outcome
- **Preconditions**: Any assumptions or preconditions (e.g., "requires non-null input", "assumes database connection is open")

#### 4.2 Input and Output

- **Input parameters**: List each parameter with its type and expected role/content
- **Return value**: Type and semantic meaning of the return value
- **Side effects**: Any modifications to object state, fields, or external systems (database, file system, etc.)

#### 4.3 Execution Flow

A step-by-step description of what happens when the method runs:
- Describe the logical flow in natural language
- Use conditional language when flow branches ("If ... then ...", "Otherwise ...")
- Include loop iterations if relevant ("For each item in the collection, ...")
- Use clear numbering or bullet points

#### 4.4 Method Call Graph

A structured representation of all method calls:
- Display as nested list or tree
- For each call, show:
  - **Target method**: Fully qualified name
  - **Source**: in-project, external library, or stdlib
  - **Parameters**: What is passed to the method
  - **Conditional**: If the call is conditional, note the condition

Example format:
```
1. [IN-PROJECT] com.example.UserRepository.findAll()
   - Returns: List<User>
   - Used to: Retrieve all users from database
   
2. [STDLIB] java.util.List.stream()
   - Converts list to stream for processing
   
   2.1. [IN-PROJECT] com.example.UserFilter.isActive(user)
        - Filters users based on activity status
        - Conditional: Called in stream filter operation
```

#### 4.5 External Dependencies

List all external library and stdlib calls made (directly or indirectly):
- **Library/Package name**
- **Methods called** (if specific enough to list)
- **Purpose** (why the method uses this external resource)

#### 4.6 Data Flow

Describe how data flows through the method:
- What inputs are transformed into what outputs
- Any intermediate data structures created
- Any state changes or object mutations

#### 4.7 Complexity and Notes

- **Cyclomatic Complexity**: Rough estimate based on decision points
- **Notable Patterns**: Design patterns used (e.g., "Strategy pattern", "Template Method")
- **Performance Considerations**: Any obvious performance implications (loops, recursion, etc.)
- **Error Handling**: How the method handles errors or edge cases (try/catch, null checks, etc.)
- **Assumptions**: Any assumptions the analysis makes (e.g., "assumes list contains valid objects")

---

## Output Format

- Write the output in the same language the user used when invoking the skill, unless they specify otherwise.
- Use Markdown with clear headings matching the sections in Phase 4.
- Be specific and concrete — reference actual class names, method names, and code logic.
- Use code formatting for Java syntax (e.g., `methodName()`, `ClassName.field`)
- For the Method Call Graph, consider using ASCII tree diagrams or indented bullet points for clarity.
- If a method call cannot be resolved, note it explicitly: "[UNRESOLVED] package.ClassName.methodName() - not found in project"

---

## Constraints

- Do **not** modify any files during analysis. This skill is read-only.
- Do **not** run the code or execute tests unless the user explicitly requests it.
- **Stop at external libraries**: When encountering calls to external libraries (Maven/Gradle dependencies), make a reasonable assumption about what the call does and do NOT attempt to find and analyze the external source code.
- **Stop at stdlib**: When encountering calls to `java.lang.*`, `java.util.*`, `java.io.*`, `javax.*`, etc., document the call and its purpose, but do not attempt to analyze the stdlib source.
- **Recursion limit**: For recursive methods, analyze only one or two levels to avoid infinite recursion. Note the recursion and its termination condition.
- **Sampling for large call graphs**: If a method's call graph is very large (>50 distinct calls), focus on the primary execution path and note that secondary/error paths are summarized.
- **Respect code boundaries**: If the target method is in a library or framework (e.g., Spring, Hibernate), analyze only the project's override/implementation, not the framework code itself.

---

## Input Examples

User might provide:
- "Analyze the `processOrder` method in `com.shop.OrderService`"
- "What does `validateEmail` do in the User class?"
- "Trace the execution of `calculateTotal` in OrderProcessor"
- File path + method name: "In `/src/main/java/service/PaymentService.java`, explain `refund`"

## Output Example Structure

```markdown
# Analysis: OrderService.processOrder()

## Method Summary
- **Name**: com.shop.OrderService.processOrder
- **Signature**: public void processOrder(Order order)
- **Purpose**: ...

## Input and Output
- **Input parameters**: ...
- **Return value**: ...
- **Side effects**: ...

## Execution Flow
1. Validate the order...
2. Calculate total...
   - If customer has discount, apply discount
3. Process payment...
4. Update inventory...
5. Send confirmation email...

## Method Call Graph
1. [IN-PROJECT] com.shop.OrderValidator.validate(order)
2. [IN-PROJECT] com.shop.PricingEngine.calculateTotal(order)
   2.1. [IN-PROJECT] com.shop.DiscountService.getDiscount(customer)
   2.2. [STDLIB] java.util.BigDecimal.multiply()
3. [IN-PROJECT] com.shop.PaymentGateway.charge(amount, paymentMethod)
   3.1. [EXTERNAL] stripe.PaymentProcessor.process()
4. [IN-PROJECT] com.shop.InventoryService.decrementStock(items)
5. [STDLIB] java.util.List.forEach()
6. [EXTERNAL] javax.mail.Transport.send()

## External Dependencies
- **stripe-java** (Stripe payment processing)
  - Used for: Credit card processing
- **javax.mail** (Java Mail API)
  - Used for: Sending confirmation emails

## Data Flow
- Order object enters method
- Order is validated and decomposed into items and payment details
- Total is calculated from items + discounts
- Payment is processed externally
- Inventory database is updated
- Confirmation message is constructed and sent

## Complexity and Notes
- **Cyclomatic Complexity**: ~3-4 (one conditional for discounts)
- **Pattern**: Orchestration pattern (coordinates multiple services)
- **Performance**: Linear in number of order items
- **Error Handling**: Assumes PaymentGateway throws exceptions on failure
```