---
title: "Business Central Development Serie - Part 2b: AI for BC Development — The Knowledge Gap That Ships to Production"
description: "Using AI to vibe-code Business Central extensions is easy. Shipping something that doesn't break your customer's ERP is not. This article is about the gap between the two."
date: 2026-03-31T00:00:00+01:00
draft: false
author: "Kamil Sacek"
categories:
- Business Central
- Architecture
- Development
tags:
- Business Central
- AI
- Architecture
- Development
- Best Practices
- AL
- PTE
- AppSource
mermaid: false
thumbnail: BCDevSeriePart02b.svg
images:
    - src: assets/BCDevSerie/BCDevSeriePart02b.svg
      alt: "Business Central Development Serie - Part 2b: AI for BC Development — The Knowledge Gap That Ships to Production"
      stretch: none
      removeBlur: false
---

> *"We are drowning not in lies — but in plausible truths."*

This article is the companion to [Part 2]({{< ref "posts/BCDevelopmentSerie-02.md" >}}) of this serie. Part 2 explains *how* to use AI tools for Business Central development — the tools, the workflow, the guidelines. This article is about something that must come before all of that: **whether you are ready to use AI for BC development responsibly.** And specifically — what you need to know to validate what AI produces, because AI will not validate itself.

## The story that started this article

I recently visited a customer's production environment and checked installed extensions. Eight small Business Central extensions, all AI-generated, all in production. Not in test environment. Just only in production.

*"It works"*, they would said. And it did. Everything looked fine. Transactions were posting, data was flowing, users were happy.

Then I looked at the code.

Every single extension had at least one silent killer. Missing `Validate` calls — field values assigned directly, skipping business logic, trigger chains, recalculations, and half of the things that make Business Central work the way it does. Filtering on fields without keys. Logic placed in `OnAfterGetRecord` triggers on pages — a classic performance and correctness trap. Eight extensions, eight sets of problems, all invisible until the day a specific combination of data, load, or process would expose them.

The scary part? The AI produced code that compiles perfectly. It passes the basic code analysis. It works in demos. It even handles the happy path in production — until it doesn't.

## The illusion of working code

AI does not lie to you. It produces code that is *plausible*. It compiles. It follows the syntax. It handles the pattern correctly in most cases. The problem is the remaining cases — and in Business Central, those cases are where your customer's financial data lives.

There is a specific trap in BC development that AI makes much more dangerous: **code that looks correct, compiles, and works — until it doesn't.**

In most development environments, a bug surfaces quickly. Wrong output, crash, test failure. In Business Central, the worst bugs are silent. They do not crash. They silently skip business logic. They silently corrupt ledger entries. They silently create performance problems that only appear under real data volume. And to fix them means much bigger cost than cost of correct development.

Here are three examples from real code — the kind of code AI can produce when it doesn't fully understand Business Central's programming model:

### Direct field assignment instead of Validate

```al
SalesLine."Unit Price" := NewPrice;
SalesLine.Modify();
```

This looks fine. It is not. The `Validate` trigger for `Unit Price` on Sales Line calculates discounts, line amount, VAT, and updates a chain of related fields. By assigning directly, you skip all of that. The price changes. Nothing else does. The line total is wrong. VAT is wrong. The order looks correct until it posts — and even then, the ledger entries may be wrong in ways that are hard to trace later.

The correct code:

```al
SalesLine.Validate("Unit Price", NewPrice);
SalesLine.Modify(true);
```

AI can produce the correct version. It can also produce the wrong version, and it will not tell you which one it chose or why. Do you know enough to check every field assignment in the generated code? Do you know when not calling the validation and triggers is CORRECT?

### Logic in `OnAfterGetRecord` on a page

```al
trigger OnAfterGetRecord()
begin
    Rec."Total Cost" := Rec."Material Cost" + Rec."Service Cost";
    Rec.Modify();
end;
```

`OnAfterGetRecord` fires for every record shown in a list. On a page showing 50 Items, this executes 50 database queries — one per Item, every time you scroll, every time the page refreshes. On a customer with 100,000 Items, this is a performance disaster. The correct approach is to update the Total Cost when Material or Service cost changes.

### Filtering on a field without a key

```al
SalesLine.SetRange("Requested Delivery Date", StartDate, EndDate);
SalesLine.FindSet();
```

This works. On a table with 100 rows it works fine. On a table with 500,000 rows it performs a full table scan on every execution. Business Central will not throw an error. The user will just experience the system as *"slow"*. Or it works "fast enough" but the load on the SQL backend is so high that other processes will struggle. A key on `Requested Delivery Date` (or a composite key including it) would turn this into an indexed lookup. AI does not automatically add keys when it adds filters. You have to know to check.

These three examples share one property: **they are not bugs AI will tell you about**. They pass compilation. They pass basic testing. They are invisible without Business Central domain knowledge.

## Standard BC libraries: AI reimplements, you pay the price

Business Central's base application contains years of accumulated business logic. Not just functionality — but *correct* functionality. Tested against edge cases in real customer environments, updated with each major release, aligned with tax laws, warehouse flows, posting chains, and ledger integrity rules. When AI doesn't know this logic exists, it reimplements it. When it reimplements it, it gets it partially right. And partially right in an ERP is often worse than wrong.

### Item Tracking

If you ask AI to implement a feature that involves items with serial or lot number tracking, and it doesn't have access to the base app source or sufficient context, it can create `Reservation Entry` records directly. The code will look reasonable. It will even work — as long as nobody tries to post a shipment, run availability checks, or transfer the items. The moment any standard posting routine touches those entries, it will either fail with a cryptic error or silently corrupt the item tracking state. Or it will work in all these processes, until someone starts to use Reservations.

The correct approach is to use standard codeunits and the standard item tracking infrastructure. These exist precisely to handle the complex interactions between reservation entries, tracking specifications, warehouse entries, and ledger entries. They are not optional shortcuts. They are the contract that keeps item tracking consistent across the system.

### JSON handling

Business Central has `JsonObject`, `JsonArray`, `JsonToken`, `JsonValue` — a complete built-in JSON library. AI can sometimes build JSON by string concatenation:

```al
JsonString := '{"name": "' + CustomerName + '", "no": "' + CustomerNo + '"}';
```

This is not just inelegant — it is a bug waiting to happen the moment a customer name contains a quote, a backslash, or a Unicode character. The built-in library handles all of this correctly and is the expected pattern. Do you know enough to verify every place JSON is constructed in the generated code?

### The general principle

Every time AI writes a function that touches a standard BC process — posting, item tracking, number assignment, approval workflows, warehouse management, manufacturing planning — you need to ask: *"Is this calling the standard BC infrastructure or reimplementing it?"* If it is reimplementing it, it is wrong. Not because the logic is necessarily incorrect at that moment, but because it will diverge from the standard behavior over time as BC evolves, and it will not integrate correctly with other extensions that rely on the standard events and interfaces. And to be able to answer this quesiton, you need to know what exists in the "standard BC".

## What you need to know to validate AI output

Vibe coding Business Central extensions responsibly is not a one-dimensional skill. It requires knowing enough across multiple domains to catch what AI gets wrong in each of them. The list below is not exhaustive — Business Central is a complex platform and there will always be areas not covered here. But it gives you an honest picture of the breadth of knowledge required.

### Architecture

Do you know when to create a new table versus extend an existing one? When to create new application and when modify existing one? If you don't, AI will make these decisions for you — and it will make them based on the simplest interpretation of the request, not the best long-term design. The [SaLi architecture articles]({{< ref "posts/SaLi_Part1.md" >}}) describe some principles that apply directly here, and you need to validate AI output against them. And do not forget about SOLID principles.

### Performance

- Are all filters applied before `FindSet()`?
- Are `SetLoadFields()` used to load only necessary fields?
- Are keys defined for every field (or field combination) used in `SetRange`/`SetFilter` to make SQL happy?
- Is `Query` used instead of nested record loops — the "loopy-loopy" code that looks fine and kills performance?
- Are page triggers (`OnAfterGetRecord`, `OnAfterGetCurrRecord`) free of database operations that run per row?
- Is `ReadIsolation` set correctly to avoid unnecessary locks on read operations?

These are not advanced topics (and definitely not complete list). They are table stakes for BC development. If you cannot check these in AI-generated code, you will deliver performance problems.

### Standard BC processes and libraries

As discussed above: can you recognize when AI is reimplementing something that already exists? Do you know the key codeunits — `ItemTrackingManagement`, `ReservationEngineMgt`, `NoSeriesMgt`, `WhseManagement`, `GenJnlPostLine`, `SalesPost`, `PurchPost`? Do you know which process to call and when? If not, AI can reinvent them — incorrectly.

### AL object design

Field numbering, object numbering, `ObsoleteState` handling, correct use of `TableRelation`, `CalcFormula`, FlowField versus regular fields — these details are invisible in demos but critical for upgrade compatibility and long-term maintainability.

### Install/Upgrade code

In many cases you need to think about existing data or new fields/tables data when installing the application. Are you able to validate, if AI correctly created upgrade code to move data when you are obsoleting some field replaced with new one?

### Locking and concurrency

Deadlocks in Business Central are not rare. They happen when multiple users or jobs access the same records in conflicting order or with incompatible lock modes. AI has no understanding of your customer's concurrent usage patterns. `ReadIsolation::ReadUncommitted` on reporting queries, `LockTable()` placement, `Commit()` placement — these require understanding of the transaction model that AI simply does not have.

### Testing

Can you tell if the generated tests actually test something meaningful? Are they isolated? Do they set up proper test data? Do they cover the error paths, not just the happy path? AI can write test codeunits that compile and run green while testing almost nothing useful. If you cannot evaluate test quality, you cannot trust the test coverage.

And to be direct: **creating an extension without automated tests is a road to hell for the future**. It is not a time-saving shortcut. It is debt that compounds with every future change, every BC version upgrade, every new developer who touches the code. AI can write the tests — but someone needs to validate that the tests are actually testing the right things.

### CI/CD and source control

If you do not have a build pipeline that compiles, runs tests, and enforces code analysis on every commit — AI-generated code is going straight to production with no gate. AI amplifies this problem: it can produce 10x more code, which means 10x more unchecked code in production if there is no automated gate. What you will do when the new version is not working? How you will fall back to previous version?

### Security and permissions

Permission sets must be correct, minimal, and explicit. `InherentPermissions` must be used appropriately. Fields containing sensitive data must have `DataClassification` set correctly. AI can produce over-permissive permission sets and ignore data classification. Neither is acceptable in a production extension.

### Telemetry and error handling

Proper `ErrorInfo` objects with action callbacks, `TryFunction` patterns, telemetry signals for key operations — these are what separate professional extensions from "customizations".

A few specific things AI can get wrong here:

- Using bare `Error()` calls instead of structured `ErrorInfo` with user-actionable messages where appropriate
- Overusing `TryFunction` as a shortcut for error handling, masking actual errors instead of handling them
- Adding no telemetry at all — leaving you with nothing to debug when something breaks in production
- Generating telemetry signals that flood Application Insights with unnecessary, non-actionable events — which not only obscures real signals but directly increases your Application Insights costs

Good telemetry is deliberate. It signals meaningful state changes and errors. It is not a log dump.

### API consumption and external integrations

When your extension calls external APIs, the code needs correct `HttpClient` usage, proper authentication (OAuth, API keys stored securely — not hardcoded), retry logic, timeout handling, bound actions where appropriate, and correct error handling for HTTP failures. AI can hardcode URLs and credentials, omit retry logic, and not handle HTTP error responses correctly. In the best case this is a maintenance problem. In the worst case it is a security vulnerability. And calling external API as part of posting? No problem for AI! But problem for end-users...

### Localization

Captions must be translatable. Reports must use AL translation files. Missing `Caption` properties, hardcoded string values, non-translatable labels — these are invisible until you deploy the extension in a language other than English. This is a particular trap for developers whose mother language is not English: AI will often generate strings in the developer's native language, producing an extension that is effectively localized to the wrong language from the start.

## The gate should be at the input, not the output

There is an important reframing of the validation problem: **trying to catch AI mistakes in code review is already too late and too expensive.** The right place to enforce quality is *before the agent writes a single line of code* — in the instructions, guidelines, guardrails, and skills you give it.

A plain AI agent with no rules is the highest-risk configuration. And it is the most common one. Someone opens GitHub Copilot Agent or Claude Code, types a requirement, and accepts whatever comes out.

The correct approach — described in detail in [Part 2]({{< ref "posts/BCDevelopmentSerie-02.md" >}}) — is to encode your quality standards into the agent's instructions: naming conventions, architecture rules, performance patterns, error handling requirements, permission set structure, event publishing expectations. When the agent follows these rules from the start, the output quality is fundamentally different from what a plain agent produces.

This does not eliminate the need for review (to learn or from any other reason, if you need it). But it shifts the review from "find all the problems" to "verify the agent followed the rules" or "haven't we forget some rule" — a much narrower and more clear task.

And create such an AI agent system is not simple job.

## It is not a one-person job

Look at the validation dimensions above. Architecture. Performance. Standard BC processes. Events. Object design. Locking. Testing. CI/CD. Security. Upgrades. Telemetry. API patterns. Localization. Each of them requires real experience to validate correctly.

No single person is expert in all of them. Not even the most experienced BC developer covers every dimension equally well.

When you use AI to write code, you are implicitly committing to validate its output across all relevant dimensions before shipping. If there are gaps in your knowledge — and there will be — those gaps become risks. And unlike a junior developer who writes code slowly and predictably, AI produces large amounts of code quickly. The surface area for validation grows faster than most people realize.

And here is a trend already visible in the market: the explosion of **"vibe-coding cleanup experts"** — developers whose primary work is fixing AI-generated extensions/applications that shipped to production without proper validation. It is a new job category created entirely by the gap between what AI produces and what production environments require. Do not create that work for someone else. Or for yourself.

Solution? "Replicate" the team structure and knowledge into the AI Agent infrastructure. And this needs time and knowledge how to do that correctly. It needs maintenance as the models are evolving. It needs monitoring to ensure the agent continues to follow the rules as it writes code. It is not a "set it and forget it" system. Can you do that? Do you have the resources to do that? If not, you are not ready to use AI for production BC development.

## When is it acceptable?

This article is not saying *never use AI for Business Central development*. It is saying: know what you are committing to when you do.

**Proof of Concept and learning**

If you are building a PoC to demonstrate a concept to a customer, or to explore feasibility, or to learn — use AI freely. The code is not going to production. The goal is speed and exploration. Just be transparent about what it is and do not let a PoC silently become a production system (it happens more often than anyone admits).

**Experienced teams with proper rules and guardrails**

If your team has the coverage to validate AI output (and to build the proper AI infrastructure) — the architecture expertise, the BC domain knowledge, the automated testing infrastructure, the CI/CD pipeline, and the agent instructions that encode your quality standards — then AI is a powerful productivity multiplier. This is the right way to use it.

**Isolated, well-defined, low-risk changes**

Adding a caption, creating a simple report extension, adding a field to a page with no business logic — these are low-risk tasks where AI produces good results and validation is straightforward. The risk is proportional to the complexity of BC business logic involved. But still, even the simples case can be done wrongly. Are you able to validate that?

**What is not acceptable**

- Code going to production without review by someone who understands BC development
- Code involving posting, item tracking, warehouse management, approval workflows, or any other complex BC process without validation by someone who knows those processes deeply
- Extensions built without any automated tests
- Any code where the author cannot explain what every generated function does and why
- Using a plain AI agent with no rules, instructions, or guardrails and treating its output as production-ready

## Self-assessment: are you ready?

Before you use AI to generate Business Central extensions for production, answer these questions honestly:

1. Can you look at an AL codeunit and identify missing `Validate` calls?
2. Do you know which standard BC codeunits should be called for item tracking, number series, posting, and warehouse management — and can you spot when AI reimplements them instead?
3. Can you design (and validate) the correct table keys for the filtering patterns used in the extension?
4. Can you identify page trigger misuse that causes per-row database queries?
5. Do you know when `ReadIsolation` should be set and what the consequences of getting it wrong are?
6. Can you recognize "loopy-loopy" code that should be rewritten as a Query?
7. Can you evaluate whether a test codeunit actually tests something meaningful?
8. Do you have a CI/CD pipeline that blocks merges on test failures or code analysis warnings?
9. Do you know how to use BC's permission model correctly — inherent permissions, data classification, minimal privilege?
10. Can you verify that JSON construction, API calls, and external integrations are implemented without hardcoded credentials and with proper error handling?
11. Do you know what BC's deprecation notices mean?
12. Have you encoded your quality standards as instructions and guardrails for your AI agent — before it writes the first line?

The answers are for you. There is no passing score published here — you know better than anyone else where your gaps are. The goal of this list is to make those gaps visible, not to judge.

There is no shame in having gaps. Everyone does. The danger is not having them — it is not knowing you have them.

## Summary

Part 2 of this serie explained *how* to use AI for Business Central development: the tools, the workflow, the guidelines, the prompting techniques. This article is about something that must come first: **the honest self-assessment of whether you are ready to validate what AI produces.**

AI is not a shortcut past expertise. It is an amplifier of expertise. In the hands of an experienced BC developer with proper guardrails and a proper process, it is a genuine productivity multiplier. In the hands of someone who doesn't yet have the domain knowledge to validate the output, it is a fast way to produce plausible-looking extensions full of silent problems.

*"We are drowning not in lies — but in plausible truths."*

The eight extensions running in production — compiling, working, apparently fine — were plausible truths. Every one of them was a risk. None of them were malicious. The AI produced what it was asked for. The problem was that nobody had the knowledge to check whether what was asked for was what was actually needed — and no guardrails were in place to guide the agent toward what was correct.

Business Central is not a generic platform where mistakes are cheap. It is an ERP system where your code runs inside your customer's financial processes. Mistakes have real consequences — wrong ledger entries, corrupted inventory, performance problems that affect entire companies during peak season. The standard of validation required is high, and it does not go down just because AI wrote the first draft.

**The gate must be at the input.** Encode your rules before the agent writes. Build your team's coverage so no validation dimension goes unchecked. Set up the infrastructure that catches problems before they reach production.

And if you are not there yet — be honest about it. Use AI for learning and PoC. Build your expertise. Find someone who can review your work with the knowledge this article describes.

The shortcut is not vibe coding. The shortcut is learning faster — with AI as your teacher and a real expert reviewing your work until you no longer need one.

Do not hesitate to contact me if you have questions, disagreements, or experiences to share. This is a topic the whole BC community is working through together, and the discussion is more valuable than any single article.
