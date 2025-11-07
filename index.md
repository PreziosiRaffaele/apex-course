# What Is Apex?
Apex is Salesforce's server-side programming language for embedding custom logic directly inside the Salesforce Platform. 
It is strongly typed, object-oriented.

**Strongly typed** means every variable must declare a specific type (for example `Account`, `Integer`, or `List<Contact>`), and the compiler enforces those types before code runs, preventing accidental misuse at runtime.

## Why Apex Exists
- Extend Salesforce beyond declarative point-and-click tools.
- Handle complex business logic, data automation, and integrations.
- Provide a consistent way to run trusted code inside Salesforce's multi-tenant infrastructure, meaning Salesforce can vet and execute customer logic safely beside other tenants' code without exposing data or destabilizing the shared runtime.

## Key Characteristics
- **Managed runtime:** Salesforce executes Apex on shared infrastructure and enforces governor limits (CPU time, heap size, SOQL/DML counts) to preserve fairness.
- **Database native:** Apex speaks SOQL/SOSL for querying data, and DML statements for inserts/updates/deletes.
- **Deployment aware:** Code moves through orgs (scratch → sandbox → production) and must include tests with ≥75% coverage before deployment.
- **Event driven:** Apex runs in specific entry contexts that the platform spins up in response to events—data changes, schedule ticks, API calls, UI requests, or automation hooks—so you architect logic around these well-defined execution points.

### Apex Entry Points
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

## When to Choose Apex
- Declarative tools (Flows, Process Builder) cannot express the branching, loops, or data volume you need.
- You must integrate with external systems using HTTP callouts or custom REST/SOAP services.
- You need reusable services, complex validation, or domain logic shared across multiple entry points.

## Comparison to Other Salesforce Tools
- **Flows:** Great for admins and straightforward automation; Apex takes over when logic becomes dense or performance sensitive.
- **Lightning Web Components:** Client-side UI that often invokes Apex classes to fetch or mutate data.
- **Platform Events:** Apex subscribers process events for near-real-time integrations and decoupled workflows.

## Minimal Apex Example
```apex
public with sharing class AccountUtility {
    public static List<Account> findTopCustomers(Integer limitSize) {
        limitSize = Math.max(1, Math.min(limitSize, 200));
        return [
            SELECT Id, Name, AnnualRevenue
            FROM Account
            ORDER BY AnnualRevenue DESC
            LIMIT :limitSize
        ];
    }
}
```
This simple class shows the Java-like syntax, SOQL query integration, and governor-friendly safeguards (bounds checking) that every Apex developer must consider.
