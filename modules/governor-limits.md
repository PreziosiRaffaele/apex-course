# Governor Limits: Respecting Shared Resources
Governor limits are the Salesforce runtime rules that keep one customer’s automation from monopolizing shared compute resources. Understanding the “why” behind these limits helps you detect risky designs even before you touch code.

## Why the Platform Enforces Limits
- **Multi-tenant fairness:** All orgs run on pooled hardware, so Salesforce caps resource usage to guarantee predictable performance across customers.
- **Defensive diagnostics:** Consistent limits reveal runaway logic early—if a Flow suddenly causes 200 SOQL queries, the exception points straight to the flawed design.
- **Declarative parity:** Limits apply to Apex, Flow, and some managed packages. A limit error often means “time to redesign,” not “blame the developer.”

## Types of Governor Limits
1. **Per-transaction limits** — Counters that reset after a transaction finishes (e.g., SOQL queries, DML statements). These are the limits you hit most often inside triggers and synchronous controllers.
2. **Per-execution context limits** — Variations for async jobs, batch execute methods, and test methods. Each context gets its own bucket, so queueing work can buy breathing room.
3. **Rolling or org-wide limits** — Daily email callout caps, concurrent long-running jobs, or streaming events. These span multiple transactions and can block deployments if ignored.

## Key Apex Limits to Monitor (Not a Full List)
- **SOQL queries (100 sync / 200 async):** Excessive record-by-record queries typically signal logic sitting inside loops.
- **DML statements (150 per transaction):** Nested updates or inserting child rows in loops spikes this fast.
- **Heap size (10 MB sync / 12 MB async):** Over-fetching fields or building giant in-memory collections can exhaust runtime memory.
- **Callouts (10 per transaction):** Integrations chained to user-driven triggers need batching or async patterns.

Whenever possible, inspect counters with the `Limits` class while debugging to identify which resource is almost exhausted.

## Reading Exercise: Spot the Risk
```apex
trigger AssetTrigger on Asset (before update) {
    for (Asset rec : Trigger.new) {
        List<Case> cases = [
            SELECT Id FROM Case WHERE AssetId = :rec.Id
        ];
        for (Case c : cases) {
            c.Status = 'Investigate';
            update c;
        }
    }
}
```
- What is this loop trying to accomplish?
- Which limit is put at risk and why?
- How could you redesign the logic so that declarative changes (like adding another lookup update) don’t multiply the problem?
