---
title: "SaLi architecture for Microsoft Dynamics 365 Business Central, part 1"
date: 2023-10-17T16:30:00+02:00
draft: false
categories:
- Business Central
- Architecture
- PTE
- AppSource
tags:
- Business Central
- Architecture
- PTE
- SaLi
- AppSource
thumbnail: SaLiArchitectureHeadline.jpeg
images: 
    - src: assets/SaLi/SaLiArchitectureHeadline.jpeg
      alt: SaLi Architecture
      stretch: none
      removeBlur: false
---
If you are developing solutions for Microsoft Dynamics 365 Business Central and you are building them from multiple applications (see [Split or not to split]( {{< ref "posts/PTEArchitecture1.md" >}})), may be you are now experiencing some problems with the dependencies between your apps, chaos made by development withouth rules. We were there too and this is example of dependency chart of one our solution:
{{< fancybox path="/assets/SaLi" file="Before.png" caption="Dependencies before SaLi" gallery="SaLi" >}}
And because my coleague [Jakub Linhart](https://www.linkedin.com/in/jakub-linhart-11925a96/?originalSubdomain=cz) is smart and tried to analyze our existing solutions to find some ways how to make them and our new solutions better, we have noticed some patterns and we tried to make some rules based on these patterns. Then I read book "Clean Architecture" from Robert C. Martin and learned about SOLID principles. And all started to give some meaning. And SaLi was born.

## SaLi

Why SaLi? Just simple because **Sa**cek and **Li**nhart.
But WHY SaLi? Because doing good architecture without rules is like trying to build high tower from bricks just by throwing the bricks on one heap.
{{< fancybox path="/assets/SaLi" file="BricksOnTheHeap.jpeg" caption="Bricks on the heap" gallery="SaLi" >}}
And SaLi is giving us some rules we can follow, to create nice solution which will be SOLID.
{{< fancybox path="/assets/SaLi" file="BricksWithSomeRules.jpeg" caption="Bricks with some rules" gallery="SaLi" >}}

### Application Groups

First thing we have noticed was, that applications we have developed, could be divided into multiple groups based on their purpose. This groupping was already mentioned in [Split or not to split]( {{< ref "posts/PTEArchitecture1.md" >}}) article. But we made some fine-tuining on them and weare now using these application groups:

#### Shared/Discrete apps

Apps providing discrete functionality or adding totally new area of functionality. These apps can live alone (or on top of "standard" functionality) and will provide functionality to the end-user which could be used directly (Discrete functionality). Another attribute of these app is, that they have data which are used by many other applications in the solution (Shared). In these apps you have mostly some Master tables, Journal tables, Ledger tables, Document tables etc.

#### Process/Automations apps

Apps of this type are interconnecting other apps, adding some automation on top of existing functionality. They are mostly not adding new processes or functionality, they are simplifying existing functionality or are "glueing" functionality of multiple separate apps. They have no meaning withouth the apps which they are extending and are implementing different processes of the company.

#### Input/Output apps

These apps are defining outer interface of the system. Are implementing APIs (input/output for other systems), changing GUI (output to display), adding Reports (output to printer/display). They should not have functionality which is not directly connected with the interface they are connecting. They are interface-specific (implementing specific interface connection, API vendor etc.).

### Layers

In next step, after defining application groups, we have found that these apps should create layers, which are defined by their dependencies. Together with SOLID principles, we can assign different attributes to the groups like:

- Shared/Discrete apps are defining critical business processes, are more abstract/universal, are stable (not changed too often), depending on less apps, more apps are depending on them
- Process/Automation apps are defining processes, are not stable as Shared/Discrete apps, could depends on more apps than depending on them
- Input/Output apps are unstable because are changed often, are depending on specific interfaces (are specific), are depending on other apps, no apps are depending on them

And this created layers which could look like this:
{{< fancybox path="/assets/SaLi" file="LayersCircle.png" caption="SaLi layers as circle" gallery="SaLi" >}}

#### Dependencies Rule

Based on SOLID principles, the dependencies could lead only from outer layer to inner layer, never oposite, because unstable apps should depends only on more stable apps. Specific apps should depends on generic apps, not critical apps should depends on more business critical apps etc.

This gives us first rule of SaLi: **"Dependency direction is given by layers into which the apps belongs. Dependency could go only from outer layer to inner layer (or from top to bottom if, see later). If needed in oposite, use the Dependency Inversion Principle to switch the dependency direction"**.

Another way how to describe the layers is like this:
{{< fancybox path="/assets/SaLi" file="Layers.png" caption="SaLi layers" gallery="SaLi" >}}

You can see that we have even the AppSource app layer and Core app layer. These are apps out of our control and we can only depend on them. But even when you are developing solution for AppSource, the layers Shared/Process/Input&Output are about your AppSource solution (if it is created from multiple apps) and the AppSource layer means other AppSource apps you can be depending on.

The arrows are marking the direction in which the dependencies could be created in between apps.

Applying this rule, and use the layers when creating application dependency chart, will result in something like this:
{{< fancybox path="/assets/SaLi" file="After.png" caption="Dependencies after SaLi" gallery="SaLi" >}}

But still, on this picture you can see arrows which are breaking the rule (connecting e.g. apps on same layer, going from bottom up etc.). Yes, these are errors in the architecture, and SaLi is helping us to see them. When we see them, we can fix them by implementing Dependency Inversion Principle (using interfaces, events...). But without SaLi they are nicely hidden and will pop up in the least appropriate time.

Another income of SaLi in this case is, that we have the layer assigned to the app stored in our metadata for every app (we are using Work Items in Azure DevOps for this) to be able to create the chart. And because this, we can implement check in DevOps pipelines which can validate if the dependencies are in correct directions and can enforce developers to create them correctly (work in progress on this).

## Next

In next article about SaLi we will look on other rules we are using.
