# Apex for Salesforce Professionals 
Course Objectives
By the end of the course, participants will:
Understand what Apex is, why and when it’s used.
Be able to read, interpret, and discuss Apex code confidently.
Know how Apex interacts with Flows, Triggers, and the database (SOQL/SOSL).
Identify when a requirement needs Apex and when it doesn’t.
Communicate effectively with developers on technical designs.

## Learning Path 
1. [Intro to Apex](modules/intro-to-apex.md) — Understand why Apex exists, how it complements declarative tools, and explore the platform runtime before attempting larger projects.
2. [Declarative vs Programmatic Tools](modules/declarative-vs-programmatic.md) — Build a repeatable decision model for when Flow, approvals, or Apex should own a requirement, and learn how to brief developers with risk-aware context.
3. [Apex Data Types](modules/apex-data-types.md) — Master scalar types, `Id`/date handling, and the differences between strong typing on `sObject` records versus the dynamic `sObject` base class.
4. [Apex Collections](modules/apex-collections.md) — Learn when to choose Lists, Sets, or Maps, and practice bulk-safe DML/SOQL patterns that scale beyond single-record thinking.
5. [Apex OOP Fundamentals](modules/apex-oop-fundamentals.md) — Apply sharing keywords, inheritance, interfaces, and enums to model business rules cleanly.
6. [Governor Limits](modules/governor-limits.md) — Diagnose and design around transactional limits so automations remain reliable under enterprise load.

## How to Use This Repo
- Follow the modules in sequence; each lesson assumes mastery of the prior concepts.
- Run every code sample in an org (scratch, sandbox, or Trailhead playground) to see how limits and types behave live.
- Capture worksheet outputs or supporting diagrams in `assets/` so the GitHub Pages build stays portable.
- Close each module by completing the exercise; these scenarios mirror stakeholder requests you’ll encounter in real projects.
