# Declarative vs. Programmatic (Apex)  
Salesforce provides a rich set of **declarative tools**—Flows, Validation Rules, Approval Processes, Assignment Rules, etc.—that allow us to design business logic without writing code.

## 1. Why Salesforce Provides Declarative Tools

Declarative tools are designed to:

- **Be easy to access**  
  Anyone with admin/consultant skills can use them without deep coding knowledge.
- **Enable rapid development**  
  Faster to configure, test, and deploy small–medium complexity logic.
- **Provide structure**  
  Built-in frameworks and patterns to solve business problems consistently.
- **Provide guardrails**  
  Reduce the chance of introducing security, performance, or logic errors.

---

## 2. Our Goals when we develop a new feature 

Regardless of the tool (declarative or programmatic), our goals are:

- **Build the right thing for the client**
- **Build it fast**
- **Prevent issues later**

### 2.1 Why build fast?

- **Faster feedback from users**  
  The sooner we deliver, the sooner we learn if we got it right.
- **Assumptions are often wrong**  
  Many features end up underused or unused. We should validate ideas early.
- **MVP mindset**  
  - Deliver a **Minimum Viable Product (MVP)** to validate the idea.  
  - An MVP is **not** a POC: it should work reliably in production, even if minimal.

### 2.2 Why prevent issues later?

- **Issues damage client trust**
- **Fixing bugs takes time away** from building new value
- **Stable foundations reduce long-term cost**

> Spending less time on building and fixing issues provides more time to focus on **building the right thing**.

---

## 3. Declarative Tools

In general, declarative tools are the **first and safest choice** when designing automations in Salesforce.

### 3.1 Strengths of declarative tools

- **Fast to build**
- **Lower risk**
- **Easier for admins to understand and maintain**
- **Guardrails and standard patterns**
- **Lower cost of change**

Typically, declarative tools:

- Offer sufficient performance for standard business processes.
- Integrate natively with core Salesforce features.
- Reduce dependency on specialized development skills.

---

## 4. Programmatic Tools (Apex & Code)

Declarative solutions are not always enough. There are scenarios where **Apex or other programmatic approaches** are the better—or only—option.

### 4.1 Strengths of programmatic tools

- **More flexibility**  
  Handle complex and edge-case logic beyond what flows or declarative tools can model clearly.
- **Better reusability**  
  Classes, methods, and frameworks can be reused across multiple features.
- **Performance and scalability**  
  Better suited for heavy processing and large data volumes.
- **Structured complexity**  
  Complex business rules can be expressed more clearly in code with good design.
- **Design patterns**  
  You can use standard patterns (e.g., Service Layer, Repository, Strategy) to build robust architecture.
- **Maintainability (for experts)**  
  For experienced developers, a well-structured Apex solution can be easier to read and extend.

### 4.2 Typical use cases for programmatic tools

Use Apex or other code-based approaches when:

- Business logic is **too complex or messy** to model with flows.
- You need **advanced error handling** or transaction control.
- You require **high-performance bulk processing**.
- Logic must be **reused across many touchpoints** (triggers, LWC, integrations, etc.).
- You need **custom integrations** with external systems.
- You’re building a **framework or shared library** for a wider solution.

---

## 5. Examples of Functionality (Flows vs. Code)

Examples of functionality that might be implemented in **flows**:

- Simple record automation (create/update related records).
- Field updates and notifications based on straightforward conditions.
- Guided user processes (screen flows) with limited branching.
- Simple approval routing based on field values.

Examples where **code** may be more appropriate:

- Complex multi-step orchestrations with branching and error recovery.
- Heavy calculation engines. 
- Reusable business services consumed by LWC, APIs, and triggers.
- High-volume data operations (e.g. nightly jobs, mass data updates).

---

## 6. There Is No Simple Rule of Thumb

There is **no single golden rule** that automatically tells you whether to choose declarative or programmatic tooling.
Since declarative tools are designed to be easy to use, they are often the **first choice** for many use cases.
Document when you choose programmatic tools.  

### 6.1 Key decision criteria

When deciding between declarative vs. programmatic, consider:

1. **Complexity**  
   - Can it be implemented clearly and safely in declarative tools?
3. **Performance**  
   - How often does it run? On how much data? In which contexts (synchronous/async)?
4. **Reusability**  
   - Will this logic be needed in multiple places or channels?
5. **Scalability**  
   - Is this likely to grow more complex over time?
6. **Governance & guardrails**  
   - Do platform limits and flow constraints help or hinder here?
7. **Risk**  
   - Which approach reduces the risk of defects and regressions?
