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
More details will be explained in the OOP module.

## Why Apex Exists
- Extend Salesforce beyond declarative point-and-click tools.
- Handle complex business logic, data automation, and integrations.
- Provide a consistent way to run trusted code inside Salesforce's multi-tenant infrastructure, meaning Salesforce can vet and execute customer logic safely beside other tenants' code without exposing data or destabilizing the shared runtime.

## Key Characteristics
- **Managed runtime:** Salesforce executes Apex on shared infrastructure and enforces governor limits (CPU time, heap size, SOQL/DML counts) to preserve fairness.
- **Database native:** Apex speaks SOQL/SOSL for querying data, and DML statements for inserts/updates/deletes.

### Database Native

SOQL can be used to fetch a list of records from the database.

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

DML statements (`insert`, `update`, `delete`, `upsert`) can be used to create, update, or delete records in the database.

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

### Apex Entry Contexts 
Each entry context is simply a doorway Salesforce opens before running Apex. Seeing the doorway explains why the code exists and which limits apply.

**DML triggers** fire automatically when records change. _Example: When a data import inserts Accounts without an Industry, a `before insert` trigger fills in “Unspecified” across every row before the transaction commits._

**Anonymous Apex** runs as soon as you click Execute in Developer Console or VS Code. _Example: An admin runs a one-time script that default the Industry with "Unspecified" value on 200 stale Accounts during a maintenance window; the script executes immediately in the admin’s session._

**Lightning / Flow invocations** execute when a component, Flow, or Process Builder calls an exposed method. _Example: An Lightning web component calls `fetchLead` to pull the current Lead’s Company and Status so the UI can render a summary; 

**Web services** expose Apex as REST or SOAP endpoints that external systems call. _Example: A warehouse management system posts to `/services/apexrest/returns` with an Order Id; the Apex REST class verifies the customer is entitled to a return, allocates replacement inventory, and writes a Case plus a custom Return__c record. 

**Asynchronous jobs** such as Queueable or Future Apex start when you enqueue them, giving the work its own transaction and fresh governor limits. _Example: After a user clicks “Generate Invoice,” the controller enqueues `InvoiceExporter`, which later wakes up, sends each invoice to an external ERP, and updates the records without keeping the user waiting._
**Scheduled Apex** starts because the platform clock reached a cron expression defined by the admin or deployment script. _Example: `MonthlyReminder` runs at 08:00 on the first business day, queries all Tasks due today, and emails the support manager a digest even though no user is online at that moment._

**Testing context** begins when an `@IsTest` method runs during `Run Tests` or deployment. _Example: `AccountTriggerTest` inserts a mock Account, asserts the trigger defaulted Industry, and then Salesforce rolls back every insert so the org stays clean._

**Email services** start when Salesforce receives an email tied to an Apex handler address. _Example: A customer sends an email to `support@yourcompany.salesforce.com`; `SupportEmailHandler` converts the subject and body into a new Case without a human agent touching the UI._

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
