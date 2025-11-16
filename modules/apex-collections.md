# Collections
In programming, collections are data structures that group multiple values into a single container so you can store, organize, and manipulate them more easily.

Data structure is a way to organize and store information.

## Apex Collections
- **List** – ordered group of elements; positions start at index `0`. Duplicates are allowed.
- **Set** – unordered group of unique elements; duplicate inserts are ignored.
- **Map** – key/value pairs; each key appears once and returns exactly one value - Think of it like a mathematical function that takes a key and returns a value.

### Micro Examples
```apex
List<String> steps = new List<String>{'Create User', 'Assign Permission Set', 'Create User'}; // We can add multiple times the same element
steps.add('Ship Laptop');
String firstStep = steps[0]; // "Create User"

Set<Id> ownerIds = new Set<Id>();
ownerIds.add(userA.Id);
ownerIds.add(userA.Id); // still one entry
Boolean containsB = ownerIds.contains(userB.Id);

Map<Id, Contact> primaryContactByAccount = new Map<Id, Contact>(); // Key is the Id of the Account, value is the Contact
primaryContactByAccount.put(acme.Id, acmePrimary);
Contact acmeContact = primaryContactByAccount.get(acme.Id);
```

## Why Collections Exist
Collections are fundamental in programming because they let you group, organize, and manage multiple pieces of data efficiently. 

## Lists 
Lists preserve insert order and accept duplicates. 
Use them to queue records for DML, for loop processing, fetch data from a SOQL query, or for ordered responses.

### Reading Exercise
```apex
List<Case> escalatedCases = [  // You can fetch a list of Cases with a SOQL query
    SELECT Id, Priority, Status
    FROM Case
    WHERE Status = 'Escalated'
];
for (Case c : escalatedCases) { // Loop through the list to process each Case
    if (c.Priority == 'High') {
        c.OwnerId = supportDirectorId;
    }
}
update escalatedCases; // Update all the Cases in the list
```

In this example you can see how to use a List to store a group of Cases, loop through them to process each one, and then update them all at once.

## Sets: Enforce Uniqueness
Sets remove duplicates automatically, which is useful whenever a requirement says “count or check each Id once.”

```apex
Set<Id> existingTeamMemberIds = new Set<Id>();
for (OpportunityTeamMember teamMember : [
    SELECT UserId FROM OpportunityTeamMember WHERE OpportunityId IN :oppIds
]) {
    existingTeamMemberIds.add(teamMember.UserId);
}

if (!existingTeamMemberIds.contains(candidateUserId)) { // Check if the candidate is already a member of one of the opportunities
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
- Customers complain they are receiving multiple time the same email. Implement a method that returns a list of unique emails, removing duplicates and empty values.
```apex
public static List<String> getUniqueValidEmails(List<String> inputEmails){
    // Your code here
}
```

-  We need to send an email to each Account owner with info about the related Contacts. Implement a method that returns a Map with the Owner Id as the key and a List of Contacts as the value.
```apex
public static Map<Id, List<Contact>> getContactsByAccount(Set<Id> accountIds)
{
    // Your code here
}
```

- Customers complain that updates on the Account with many Contacts are slow and sometimes fail with a timeout error.
Review the code below and explain why the error occurs. Refactor the code to avoid the timeout error.

```apex
trigger AccountTrigger on Account (before update) {
    for (Account acc : Trigger.new) {
        List<Contact> contacts = [SELECT Id FROM Contact WHERE AccountId = :acc.Id];

        for (Contact c : contacts) {
            c.Country__c = acc.Country__c;
            if (acc.Country__c == 'PL' || acc.Country__c == 'IT') {
                c.GDPR_Region__c = 'EU';
            }else {
                c.GDPR_Region__c = 'Non-EU';
            }
            update c;
        }
    }
}
```
