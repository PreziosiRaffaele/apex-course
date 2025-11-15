# What Is Apex?
Apex is Salesforce's server-side programming language for embedding custom logic directly inside the Salesforce Platform. 
- **It is strongly typed.**
- **It is object-oriented.**

## Strongly Typed 
**Strongly typed** means every variable must declare a specific type (for example `Account`, `Integer`, or `List<Contact>`), and the compiler enforces those types before code runs, preventing accidental misuse at runtime.

Apex example (strongly typed):

```apex
Integer qty = 10;
qty = 'ten'; // compile-time error: string literal is not an Integer

String name = 'Acme';
name = 1234; // compile-time error: string literal is not a String
```

JavaScript example (not strongly typed):

```javascript
let qty = 10;
qty = 'ten'; // allowed
```

Because Apex checks types the moment you save or deploy, these mismatches surface early, while JavaScript defers them until the logic actually executes.

### Object-Oriented 
It is programming paradigm that organizes software design around objects, that are instances of classes.
Based of the four pillars: encapsulation, inheritance, polymorphism, and abstraction.

## Why Apex Exists
- Extend Salesforce beyond declarative point-and-click tools.
- Handle complex business logic, data automation, and integrations.
- Provide a consistent way to run trusted code inside Salesforce's multi-tenant infrastructure, meaning Salesforce can vet and execute customer logic safely beside other tenants' code without exposing data or destabilizing the shared runtime.

## Key Characteristics
- **Managed runtime:** Salesforce executes Apex on shared infrastructure and enforces governor limits (CPU time, heap size, SOQL/DML counts) to preserve fairness.
- **Database native:** Apex speaks SOQL/SOSL for querying data, and DML statements for inserts/updates/deletes.

### How Apex Talks to Salesforce Data
Before Apex can read or save records, it needs to know *which* object the record belongs to. An `Account` variable represents one CRM row; a `List<Account>` holds several at once.

#### Reading with SOQL 
```apex
List<Account> unscoredAccounts = [
    SELECT Id, Name, AnnualRevenue
    FROM Account
    WHERE AnnualRevenue = null
    LIMIT 50
];

System.debug('Accounts missing revenue: ' + unscoredAccounts.size());
```

#### Writing with DML 

```apex
Account newCustomer = new Account(
    Name = 'Universal Paper Trails',
    Industry = 'Manufacturing'
);
insert newCustomer; // Saves the record and assigns an Id.

for (Account acct : unscoredAccounts) {
    acct.AnnualRevenue = 0; // Placeholder until Sales supplies the real figure.
}
update unscoredAccounts; // Commits the edits as one bulk operation.
```

- Create records in memory, then issue DML to save them.
- Looping through queried records lets you interpret existing data and prepare precise updates.

#### Spot the Bad Pattern
```apex
List<Account> everyAccountEver = [SELECT Id FROM Account];
```

This innocent line tries to pull every Account in the org. A simple declarative change (like a mass import) would make it exceed the “50,000 rows per transaction” limit and the trigger would fail. Good Apex always narrows its queries to just the rows it needs.

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
