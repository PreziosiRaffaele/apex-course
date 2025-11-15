# Apex Control Flow Structures
Control flow Structure is the set of instructions that tells Apex **which** statements run.

## Types 
- Conditional: `if`, `else if`, `else`
- Looping: `for`, `while`, `do-while`
- Breaking: `break`, `continue`
- Exceptions: `try`, `catch`, `finally`

## Branching With `if`, `else if`, and `else`
```apex
if (opp.Amount >= 1000000) {
    opp.StageName = 'Executive Review';
} else if (opp.Amount >= 250000) {
    opp.StageName = 'Regional Review';
} else {
    opp.StageName = 'Standard Process';
}
```
Ask yourself:
- What happens to an Opportunity at $500k?
- What happens at $50k?
- How many times does each branch run inside a trigger that processes 200 Opportunities?

## Switch Statements for Controlled Lists
`switch` is best when outcomes map cleanly to specific values:

```apex
switch on caseRecord.Origin {
    when 'Email' { caseRecord.Priority = 'Low'; }
    when 'Phone' { caseRecord.Priority = 'High'; }
    when else    { caseRecord.Priority = 'Medium'; }
}
```

Reading exercise: if a new Origin value of “Web Form” appears and no developer updates the code, which branch will fire? Why does that fallback matter during requirement reviews?

## Loop Structures
- **Enhanced `for (Type item : collection)`** – Expect this when iterating through `Trigger.new` or a collection list. 
- **Indexed `for (Integer i = 0; i < limit; i++)`** – Shows up when counting operations or building child lists. 
- **`while (condition)` / `do { ... } while (condition);`** – Rare in day-to-day business logic; often signals polling or error handling. 

### Examples
**Enhanced `for` (collection)**
```apex
for (Contact c : Trigger.new) {
    if (c.Email == null) {
        c.Email_Opt_In__c = false;
    }
}
```
Ask: how many Contacts flow through this loop when 200 rows are updated from Data Loader?

**Indexed `for` (counter-controlled)**
```apex
for (Integer i = 0; i < pendingTasks.size(); i++) {
    Task t = pendingTasks[i];
    if (i >= 10) break; // safety valve for notifications
    notifyOwner(t.OwnerId, t.Subject);
}
```
Ask: what prevents this from sending hundreds of notifications, and what happens when `pendingTasks` has fewer than 10 entries?

**`while` loop**
```apex
Integer attempts = 0;
while (syncQueue.isReady() && attempts < 3) {
    processNextJob();
    attempts++;
}
```
Ask: which condition stops the loop first—queue readiness or the attempts counter?

**`do-while` loop**
```apex
Integer batchNumber = 0;
List<Account> batch;
do {
    batch = getBatch(batchNumber);
    summarize(batch);
    batchNumber++;
} while (!batch.isEmpty());
```
Ask: why is it important that the first batch runs even if `getBatch(0)` returns zero rows?

**Bad pattern to call out:** performing SOQL or DML inside nested loops. Explain why: each pass consumes more limits than necessary.

### Exercise: Loop Remix Challenge
Give learners this base scenario: “Mark every Task in `pendingTasks` as `Ready_to_Send__c = true` until you hit 25 records.” Ask them to implement the exact same behavior three times:
1. Enhanced `for` loop across the list.
2. Indexed `for` loop using `i` and `pendingTasks[i]`.
3. `while` loop that tracks its own counter.

Debrief by comparing which version is easiest to read, which exposes index math mistakes, and why each structure still needs a clear exit condition to stay governor-safe.

### `continue` vs `break`
- `continue` skips the rest of the current loop iteration and moves to the next record. Use it when certain rows need special handling and should not fall through to the remaining statements.
- `break` exits the loop entirely. Use it when a condition means “stop looping for good,” such as finding the first matching record or protecting against runaway notifications.

Example:
```apex
for (Case c : Trigger.new) {
    if (c.Status == 'Closed') {
        continue; // nothing else to do for closed cases
    }
    if (followUpsSent >= 50) {
        break; // guardrail to stop mass reminders
    }
    sendReminder(c.OwnerId);
    followUpsSent++;
}
```
Ask: in this snippet, how many closed cases still evaluate the second `if` statement? What happens to cases after the 50th reminder?

### `while` vs `do-while`
- `while (condition) { ... }` checks the condition **first**, so the body might never run. Use when you only proceed if the rule is already true.
- `do { ... } while (condition);` runs the body **at least once** and checks the condition afterward. Use for tasks where you must process the first batch before deciding whether to continue (e.g., pagination).

Reading exercise: when monitoring a queue for work, which loop keeps a job from running if there is nothing to do? Which loop ensures one pass happens even if the queue starts empty?
### Code Review Example: `while` Without an Exit
```apex
while (caseRecord.Approval_Status__c != 'Approved') {
    sendReminder(caseRecord.OwnerId);
    // Approval_Status__c never changes in this loop,
    // so the condition stays true forever.
}
```
- Which governor limits (CPU time, DML, emails) could this unbounded loop hit?

## Control Flow and Declarative Changes
- Adding a new Flow that updates a field might cause an Apex `if` condition to evaluate differently, activating branches that were rarely hit.
- Changing a picklist value forces you to verify that every `switch` or `if` comparison knows about the new option; otherwise it falls into a default path that may be wrong.

## Guided Reading Exercise
```apex
for (Case c : Trigger.new) {
    if (c.Type == 'Outage') {
        c.OwnerId = escalationQueue.Id;
        continue;
    }
    if (c.LastEscalationDate__c == null) {
        c.LastEscalationDate__c = System.today();
    } else if (!c.IsClosed && c.Age__c > 5) {
        c.OwnerId = backlogTeam.Id;
    }
}
```
Discuss:
1. Why does the developer use `continue` in the first `if` block?
2. Which branch will run when the Type is blank, the case is still open, and `Age__c` equals 7?
3. What happens if an admin adds automation that populates `LastEscalationDate__c` earlier in the lifecycle?
