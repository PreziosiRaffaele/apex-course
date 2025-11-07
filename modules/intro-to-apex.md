# What Is Apex?
Apex is Salesforce's server-side programming language for embedding custom logic directly inside the Salesforce Platform. 
- **It is strongly typed.**
- **It is object-oriented.**

**Strongly typed** means every variable must declare a specific type (for example `Account`, `Integer`, or `List<Contact>`), and the compiler enforces those types before code runs, preventing accidental misuse at runtime.

## Why Apex Exists
- Extend Salesforce beyond declarative point-and-click tools.
- Handle complex business logic, data automation, and integrations.
- Provide a consistent way to run trusted code inside Salesforce's multi-tenant infrastructure, meaning Salesforce can vet and execute customer logic safely beside other tenants' code without exposing data or destabilizing the shared runtime.

## Key Characteristics
- **Managed runtime:** Salesforce executes Apex on shared infrastructure and enforces governor limits (CPU time, heap size, SOQL/DML counts) to preserve fairness.
- **Database native:** Apex speaks SOQL/SOSL for querying data, and DML statements for inserts/updates/deletes.

### Apex Entry Contexts 
- **DML triggers:** `before`/`after` triggers on objects plus Platform Event and Change Data Capture triggers.
- **Anonymous Apex:** One-off scripts executed from Developer Console, VS Code, or tooling APIs.
- **Lightning / Flow invocations:** Lightning Web Components, Aura, Flow invocable actions, and Process Builder call Apex methods marked `@AuraEnabled` or `@InvocableMethod`.
- **Web services:** Custom REST (`@RestResource`) and SOAP (`webService` keyword) endpoints responding to external API calls.
- **Asynchronous jobs:** Queueable, Future, Batch Apex, and Continuations for long-running or parallelizable work.
- **Scheduled Apex:** Cron-like jobs that fire Apex classes implementing `Schedulable`.
- **Testing context:** `@IsTest` methods run in isolated transactions to validate behavior and support deployments.
- **Email services:** Apex classes implementing `Messaging.InboundEmailHandler` process inbound email.

## How Apex Runs
1. **Compilation:** When saved, Apex compiles into bytecode stored in the org's metadata.
2. **Execution contexts:** The platform spins up contexts for triggers, anonymous apex, batch/scheduled jobs, Queueable/Future methods, and REST/SOAP endpoints.
3. **Transactions:** Each context wraps database work in a transaction; unhandled exceptions roll back all DML performed in that context.
4. **Limits check:** The runtime stops execution if governor limits are exceeded.

### Apex Transactions in Detail
- **Atomic unit of work:** A transaction begins the moment Salesforce starts executing Apex for a given entry point (trigger fire, REST call, `Queueable.execute`, test method, etc.). Every DML statement, SOQL query, callout, and limit counter lives inside that atomic boundary. When the context finishes without unhandled exceptions and without exceeding governor limits, Salesforce commits every pending DML; otherwise it rolls everything back.
- **Isolated limit bucket:** Governor limits such as "100 SOQL queries" or "10 seconds CPU" are scoped to the current transaction. Synchronous logic shares one bucket, but each asynchronous execution (`@future`, Queueable, Batch Apex start/execute/finish, Scheduled Apex) gets its own fresh bucket because each runs in its own transaction. Offloading heavy work to async contexts is therefore a common strategy to avoid limit pressure in user-facing transactions.
- **Lifecycle summary:** 
  1. Platform allocates a context and initializes limit counters.
  2. Apex executes line by line, performing DML, queries, and callouts.
  3. Execution either completes successfully, hits an unhandled exception, or exceeds a limit.
  4. Salesforce commits or rolls back all pending work.
