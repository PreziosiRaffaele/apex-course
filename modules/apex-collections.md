# Collections
In programming, collections are data structures that group multiple values into a single container so you can store, organize, and manipulate them more easily.

Data structure is a way to organize and store information.

## Apex Collections
- **List** – ordered group of elements; positions start at index `0`. Duplicates are allowed.
- **Set** – unordered group of unique elements; duplicate inserts are ignored.
- **Map** – key/value pairs; each key appears once and returns exactly one value.
Think of it like a mathematical function that takes a key and returns a value.

### Micro Examples
```apex
List<String> steps = new List<String>{'Create User', 'Assign Permission Set', 'Create User'};
steps.add('Ship Laptop');
String firstStep = steps[0]; // "Create User"

Set<Id> ownerIds = new Set<Id>();
ownerIds.add(userA.Id);
ownerIds.add(userA.Id); // still one entry
Boolean containsB = ownerIds.contains(userB.Id);

Map<Id, Contact> primaryContactByAccount = new Map<Id, Contact>();
primaryContactByAccount.put(acme.Id, acmePrimary);
Contact acmeContact = primaryContactByAccount.get(acme.Id);
```

## Why Collections Exist
Multi-record events (data loads, Flow updates, bulk API) send many records through the same transaction. Collections make that safe because:
- Governor limits count all statements per transaction, so queries and DML must already expect batches, not single rows.
- Business rules (“reject any Lead with a missing Email”) need a structure that holds every Lead until a final decision is made.

## Lists: Run Statements in Order
Lists preserve insert order and accept duplicates. Use them to queue records for DML, for loop processing, or for ordered responses.

### Reading Exercise
```apex
List<Case> escalatedCases = [
    SELECT Id, Priority, Status
    FROM Case
    WHERE Status = 'Escalated'
];
for (Case c : escalatedCases) {
    if (c.Priority == 'High') {
        c.OwnerId = supportDirectorId;
    }
}
update escalatedCases;
```
Questions:
1. Why does the query return a List instead of a single Case?
2. What happens to the second `High` priority Case if the first one is reassigned?
3. Which lines prepare data, and which line performs the DML?

Key takeaway: the last `update` statement reuses the entire List, so every Case in the batch gets the same outcome.

## Sets: Enforce Uniqueness
Sets remove duplicates automatically, which is useful whenever a requirement says “count or check each Id once.”

```apex
Set<Id> existingTeamMemberIds = new Set<Id>();
for (OpportunityTeamMember teamMember : [
    SELECT UserId FROM OpportunityTeamMember WHERE OpportunityId IN :oppIds
]) {
    existingTeamMemberIds.add(teamMember.UserId);
}

if (!existingTeamMemberIds.contains(candidateUserId)) {
    // safe to add a new team member
}

```

## Maps: Connect Related Records
Maps associate each key with a value, most often `Id -> sObject`. They prevent extra queries and nested loops when enriching data.

```apex
Map<Id, Account> accountByOwner = new Map<Id, Account>([
    SELECT Id, OwnerId, Customer_Score__c FROM Account WHERE OwnerId IN :ownerIds
]);
```

Reading prompts:
- Which Accounts are guaranteed to exist in `accountByOwner`?
- What happens when `accountByOwner.get(missingOwnerId)` runs?
- How can a non-developer confirm the query criteria? (Hint: build a list view that mirrors the WHERE clause.)

## “Bad Code” Example and the Fix
```apex
for (Case c : Trigger.new) {
    Case escalation = [
        SELECT Id, Status FROM Case WHERE Id = :c.Id
    ];
    if (escalation.Status == 'Escalated') {
        escalateIndividualCase(escalation);
    }
}
```
- **Issue:** The SOQL runs once per record because the query sits inside the loop.
- **Impact:** The code passes tests with one Case but fails when 200 Cases save together; governor limits stop the transaction and escalations never run.
- **Repair plan:** Query the Cases once into a Map keyed by Id, iterate over `Trigger.new`, and reference the Map values inside the loop.

Pull-request review prompt: **“Where are we storing the query results so we reuse them?”** A solid answer mentions Lists, Sets, or Maps.

## Collections and Declarative Changes
Declarative automation can increase the number of records your Apex receives:
- A Flow that now updates two child objects per parent doubles the size of Lists feeding a trigger.
- Duplicate Rules or Validation Rules may block DML for some entries, so confirm the Lists passed to `insert` or `update` already expect partial success.
- Omni-Channel, bulk API, or Integration User imports can create long batches. Verify the Apex paths through those objects rely on Lists and Sets instead of single-record assumptions.

## Quick Reference
- **List** — ordered, duplicates allowed, index-based access, ideal for batch DML.
- **Set** — unordered, unique values, fast membership checks, ideal for de-duplication.
- **Map** — key/value pairs, one lookup per key, ideal for joining related data.

Typical pattern: gather Ids in a Set → query using that Set → store the results in a Map → update the final List.

## Practice Scenario
QA reports that “Only some Contacts lost their `VIP_Flag__c` when an Account downgraded.” The developer shared this code:

```apex
if (oldAccount.Rating == 'Hot' && newAccount.Rating != 'Hot') {
    Contact vip = [SELECT Id, VIP_Flag__c FROM Contact WHERE AccountId = :newAccount.Id];
    vip.VIP_Flag__c = false;
    update vip;
}
```

Evaluate with these questions:
1. What happens if the Account has 30 Contacts?
2. How would a List of Contacts change the result?
3. Should we collect Contact Ids in a Set first to avoid reprocessing the same records elsewhere?

## Exercise: Trace the Data Flow
1. Map the steps “Marketing uploads Leads” → “Trigger validates duplicates” → “Trigger updates `Validation_Status__c`.” Label which collection type sits in each step.
2. Review one Apex trigger in your org and highlight the line that converts a Set of Ids into a Map of records. Explain in plain language why that prevents extra queries.
3. Explain the data flow to a teammate who works primarily in declarative tools. If they follow the story without touching the code, you have described collections clearly.
