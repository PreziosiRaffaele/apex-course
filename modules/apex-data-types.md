# Apex Data Types
Data types answer the question “what kind of information is this?” 

## Why Data Types Matter
- They protect automations: Apex enforces these types at compile time so mistakes are caught early.
- They communicate intent: when you read `Boolean needsFollowUp` you immediately know it is just a yes/no flag.

Think about a spreadsheet—each column expects specific content (number, date, text). Apex simply makes that contract explicit in code.

## Your First Apex Program: Hello World
A “Hello World” program is traditionally the smallest possible script that proves your environment runs code. In Apex you can paste the lines below into the **Execute Anonymous** window (Developer Console ▸ Debug ▸ Open Execute Anonymous Window) and click **Execute**.

```apex
String message = 'Hello, World! Welcome to Apex.';
System.debug(message);
```

### What `System.debug` Shows You
When Apex runs, Salesforce produces a short transcript of everything that happened. After you click **Execute**, open the newest entry in the Logs panel and you will see every `System.debug()` message. 

### Statements, Declarations, and Initialization
- A **statement** is a complete instruction that Apex can execute, usually ending with a semicolon. Example: `System.debug(message);`.
- A **declaration** introduces a new variable by naming its data type and variable name so Apex reserves memory for that kind of information. Example: `String message;`.
- **Initialization** assigns the first value to a variable, often in the same line as the declaration: `String message = 'Hello, World!';`. You can also declare first and initialize later in another statement.

## Primitive Types in Apex
Languages often call these “primitive” types because they are the most basic building blocks. Everything else in the language eventually relies on these primitives to describe the information inside a record or calculation.

**Apex primitives you will encounter:**
- `Boolean` — true or false values. Stored as a single bit (binary digit), so it represents 2 combinations (0 = false, 1 = true).
- `Integer` — 32-bit whole numbers, giving 2^32 (about 4.29 billion) (-2,147,483,648 to 2,147,483,647) possible values; useful for counts.
- `Long` — 64-bit whole numbers for even larger counts (2^64 combinations).
- `Decimal` and `Double` — numbers with digits after the decimal point (use Decimal for currency). They are stored using a combination of bits for the sign, digits, and scale, so they do not have a simple 2^n range, but still originate from binary storage. Double use floating-point numbers, which are stored in binary as a combination of bits for the sign, digits, and scale. They are not precise as decimals, but they are more compact than decimals.
- `String` — text of any length.
- `Id` — 15/18-character Salesforce record identifiers.
- `Date`, `Datetime`, `Time` — calendar-only, date+time, or clock-only values.
- `Blob` — raw bytes, typically used for files or encoded data (mentioned here for completeness; most business logic won’t touch it).

All data is saved as binary; what changes is the way it is interpreted.

> **Bit refresher:** A bit is the smallest possible unit of information in computing and can only be 0 or 1. The number of different values a primitive can represent is `2^(number of bits)`. For example, 32-bit Integers have `2^32` combinations.

## Exercise
Run your first "Hello, World!" script and check the debug log to see the message.
