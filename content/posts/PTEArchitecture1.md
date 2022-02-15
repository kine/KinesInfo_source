---
title: "Architecture of PTE - Split or not to Split?"
date: 2021-11-18T16:51:00+01:00
draft: false
categories:
- Business Central
- Architecture
- PTE
tags:
- Business Central
- Architecture
- PTE
---

On Directions EMEA in Milan I delivered session about Architecture of PTE for Business Central. I was trying to find some rules when or why you should or shouldn't 
split your PTE into multiple apps. Now I will try to catch this into this article.

## Split or not to split

Some partners are putting everything into one app per customer. Some are splitting all to separate apps based on different rules like per process, per area etc.
Our company is going through the way of splitting, rather than not splitting, because on the beginning we wanted to have possibility to re-use the "tiny apps" for others if needed.
But the result? Here is the picture (taken from real live customer project):

{{< fancybox path="/assets/" file="DependencyGraph.png" caption="Real world apps dependencies example" gallery="PTEArchitecture1" >}}

As you can see, some apps are "self standing". This is what we wanted to have. But most of them are connected with others. That's something we didn't want. What does it mean? They are not re-usable, if you do not want to re-use even the dependencies. It means you can reuse only the "leftmost" apps. It makes most of the apps non-reusable without refactoring.

But the question stays: **Split or not to split?**

## Types of PTE apps

based on the chart, I made some analysis and found these sets of apps based on names of the apps:

- *TOOLS/LIBRARIES* - "Record links", "Control Focus", "Barcodes", "PDF Signature"
- *IMPORTS/EXPORTS* - "Data Migration", "Salary Import Connector", "Banking"
- *CONNECTORS* - "Manufacturing API", "Laboratory Integration", "NAV Web Service Adapter"
- *SIMPLE PROCESSES* - "VAT Posting Date Change", "Standard Cost Management", "Approval", "On Hold"
- *COMPLEX PROCESSES OR AREAS* - "Service Acceptance", "Spare Parts", "Overhead Material", "FA Inventory", "Price Security", "Manufacturing", "Report Pack"…
- *EXTENSIONS (CUSTOMIZATIONS)* - "Handling Unit Extension", "Continia Extension", "Small modifications", „Intrastat Extension"

We can define some properties of each type, which could help us to identify them:
- *TOOLS/LIBRARIES* - Adding functionality which is used through whole system, independent on specific processes/areas
- *IMPORTS/EXPORTS* - working with files or other data sources
- *CONNECTORS* - connecting with other systems (through API - as client or as server) or between extensions
- *SIMPLE PROCESSES* - mostly adding some small tasks and processes, limited schema changes (not many fields and tables)
- *COMPLEX PROCESSES or AREAS* - creating new processes or whole areas, new tables, pages…
- *EXTENSIONS (CUSTOMIZATIONS)* - are modifying app which is in responsibility of someone else (3rd part, different product group etc.)

And based on their properties, we can divide them into two groups:

- **Not Problematic**
  - *TOOLS* - mostly depending on standard only (lowest level dependencies)
  - *IMPORTS/EXPORTS* - low probability of depending apps, rather depends on other (similar to CONNECTORS), sometimes need to be OnPrem if we are OnPrem
  - *CONNECTORS* - should be top (or bottom) app in the tree
- **Problematic**
  - *SIMPLE PROCESSES* - you do not know if it will not grow to be complex
  - *COMPLEX PROCESSES or AREAS* - hard to split the processes correctly and solve the dependencies when cyclic
  - *EXTENSIONS (CUSTOMIZATIONS)* - could have depending apps, could grow to complex processes

But still: **Split or not to split?**

## Differences

Ok, previous classification is not helping us to answer the question. We should look at the
 CONs and PROs of the splitting the PTE into separate Apps:

### Splitting

- **PRO**:
  - Easier parallel development (development split to more apps)
  - Clear responsibilities
  - Simpler/Smaller apps
  - Simpler maintenance
  - Re-use (really?)
  - List of functionalities
  - Possibility to „uninstall" customization when not needed (really?)
- **CONS**:
  - Dependency „hell" (solvable)
  - Architecture decisions are more complex
  - Performance (solvable)
  - Release complexity (solvable)
  - Cost of maintenance per app (partially solvable)

We can discuss if the apps are re-usable (we already touched this). But most of the CONs are solvable by using correct tools (partial records etc.) and automatic processes (release pipelines, powershell etc.).

### All-in-one

- **PRO**:
  - Simplified Architecture
  - Simplified release to Customer
  - Cost of maintenance per app
  - Performance (! not because technical issues, but because mindset of the developers - *"it is in one app, why bother with performance?"*)
- **CONS**:
  - Developers conflicts (solvable)
  - Release Planning (solvable)
  - Installation times (? in history, deploying big app took more time)
  - Impossible to re-use without refactoring
  - What is inside (solvable)
  - Shared responsibilities

And again, you can see that most of the CONs could be solved by tools (AL Object ID Ninja, documentation etc.) and processes (Azure DevOps planning etc.)

Still: **Split or not to split?**

### Comparing

Ok, compare the PROs and CONs of both methods when we remove the solvable items:

{{< fancybox path="/assets/" file="SplitProCons.png" caption="PROs and CONs comparison" gallery="PTEArchitecture1" >}}

Items I think are most interesting are in **bold**.

All depends on priorities you have in your company or team. If your priority is e.g. "Simpler maintenance" and "Clear responsibility"
and you sacrifice the "easy architecture decisions", go and split the apps.

If your priority is simpler "architecture decisions" and you sacrifice "shared responsibility" (who is responsible for the content of big, all-in-one app?), go and do not split.

Ok, does it mean **split or not to split?**

## SMART Architecture

For me, both ways are valid. But you need to do it SMART.

- **S**IMPLE
- **M**ANAGABLE
- **A**RCHITECTURE
- **R**EACHING
- **T**ARGET

And to do it, you need to know what the target is for you, your team, your company, your customers.

What it could mean? For example this:
- Keep TOOL/LIBRARIES and CONNECTOR apps separate
- Use Dependency Inversion principles (interfaces etc.) where you want to "reverse" the dependency
- Try to keep the responsibilities clear. Boundary of App is boundary of responsibility. Public interface of App is contract between apps.
- When you need to depend on new App (3rd party etc.), try to create CONNECTOR as separate app to prevent adding the dependency into the "core" app if possible.
- When you need to switch the app to „OnPrem", create CONNECTOR (working with local files etc.) - leave rest of the logic in „Cloud" (**and best is to not use OnPrem at all - it will cost you, literally**)
- Content of the app must be in line with app name. If the change is not in line with the app name, may be it should be somewhere else. Choose wise the name of the app.

## Conclusion

May be you have still questions, what is THE BEST way. But I think there is no silver bullet for this. The answer could be different, depending on who you ask.
But if you already go some way, do not switch just because someone tells you that it is better in another way. May be he/she is solving different problems than you
and your solution in your situation is the best for you.

I hope that this article helps someone to think about the architecture of Business Central PTEs from different angles, and I am open to discussion about the architecture at all.
What I see now as biggest gap in our community, is lack of architects and discussion about the architecture itself. And this my session and article is for me starter for the discussion
we will have next years.

Be safe! See you online/onprem!
