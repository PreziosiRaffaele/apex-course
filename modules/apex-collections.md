# Apex Collections
Collections let Apex handle many records at once, keeping code bulk-safe and governor-friendly. This module covers when to pick `List`, `Set`, or `Map`, plus patterns to pair with SOQL/DML.

## Lists for Ordered Work
- Maintain insertion order and allow duplicates.
- Best for batching DML or building payloads.

```apex
List<Account> accounts = [SELECT Id, Name FROM Account WHERE Industry = 'Manufacturing'];
for (Account acct : accounts) {
    acct.Description = 'Targeted for Q3 upsell';
}
update accounts;
```

## Sets for Uniqueness and Query Filters
- Automatically prevent duplicates.
- Lightning-fast containment checks (`mySet.contains(id)` is O(1)).

```apex
Set<Id> contactOwnerIds = new Set<Id>();
for (Contact c : [SELECT OwnerId FROM Contact WHERE LastActivityDate = LAST_N_DAYS:30]) {
    contactOwnerIds.add(c.OwnerId);
}
List<User> owners = [SELECT Id, Name FROM User WHERE Id IN :contactOwnerIds];
```

## Maps for Lookups
- Associate keys to values (often `Id` → `sObject`).
- Prevents nested loops when enriching data.

```apex
Map<Id, Opportunity> oppById = new Map<Id, Opportunity>([
    SELECT Id, StageName, Amount FROM Opportunity WHERE StageName = 'Negotiation'
]);

List<Quote> quotes = [SELECT Id, OpportunityId FROM Quote WHERE OpportunityId IN :oppById.keySet()];
for (Quote q : quotes) {
    Opportunity parentOpp = oppById.get(q.OpportunityId);
    if (parentOpp != null) {
        q.Description = 'Mirror opp amount: ' + parentOpp.Amount;
    }
}
update quotes;
```

## Bulkification Pattern
1. Gather record Ids into a `Set<Id>`.
2. Query related data once.
3. Use a `Map` to enrich child records without extra SOQL.
4. Perform DML on a `List` to keep ordering deterministic.

## Exercise: Lead De-duplication Helper
Build a `LeadDeduper` class with one method `public static List<Lead> removeDuplicates(List<Lead> incoming)` that:
- Uses a `Set<String>` to track composite keys (email + company).
- Returns only the unique `Lead` records ready for insert.
- Logs duplicates via `System.debug` so admins can report on them.

Provide an anonymous Apex snippet that seeds three duplicate leads, passes them to the helper, and shows only unique leads being inserted.
