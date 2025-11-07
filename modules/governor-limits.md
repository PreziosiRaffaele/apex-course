# Governor Limits and Transactions
Salesforce enforces runtime limits to keep shared infrastructure stable. This module teaches you how to detect, respect, and design around those limits before deploying automation.

## Transaction Basics
Every trigger, Flow-invoked Apex, REST call, or async job runs inside a transaction. Limits reset between asynchronous executions, so queueing work strategically can avoid user-facing errors.

## Common Limits to Monitor
- 100 synchronous SOQL queries (200 for async)
- 150 DML statements
- 10 MB synchronous heap size (12 MB async)
- 10 callouts per transaction (named credentials vary)

Use the `Limits` class or `System.debug(Limits.getDmlRows());` to inspect counters during execution.

## Bulkification Template
1. Collect triggering record Ids (`Set<Id>`).
2. Query related data once.
3. Populate a `Map` for constant-time lookups.
4. Iterate through the trigger context and update in-memory records.
5. Perform DML outside loops.

```apex
trigger OpportunityTrigger on Opportunity (before insert, before update) {
    Set<Id> accountIds = new Set<Id>();
    for (Opportunity opp : Trigger.new) {
        if (opp.AccountId != null) {
            accountIds.add(opp.AccountId);
        }
    }
    Map<Id, Account> accounts = new Map<Id, Account>([
        SELECT Id, Industry FROM Account WHERE Id IN :accountIds
    ]);

    for (Opportunity opp : Trigger.new) {
        Account parent = accounts.get(opp.AccountId);
        if (parent != null && parent.Industry == 'Healthcare') {
            opp.StageName = 'Needs Compliance Review';
        }
    }
}
```

## Defensive Patterns
- **Queueable chaining:** move heavy recalculations into async jobs to get a fresh limits bucket.
- **Platform Cache or Custom Metadata:** store configuration outside triggers to avoid describe calls.
- **Aggregate queries:** reduce rows processed when summarizing data.

## Exercise: Limit-Safe Case Reassignment
Refactor an imaginary `Case` trigger that currently performs SOQL/DML inside loops. Your new design should:
1. Collect all `OwnerId` values and query related queue memberships once.
2. Use a `Map<Id, Boolean>` to determine whether each owner is a queue.
3. Reassign Cases owned by inactive queues to a fallback user without exceeding SOQL or DML limits.

Deliverable: trigger (or handler class) plus an explanation of how many SOQL and DML statements run in the worst-case scenario. Include an anonymous Apex test script showing the trigger handling ≥200 records without errors.
