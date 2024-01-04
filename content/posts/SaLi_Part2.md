---
title: "SaLi architecture for Microsoft Dynamics 365 Business Central, part 2"
date: 2024-01-04T15:00:00+01:00
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
See previous part first - [SaLi architecture for Microsoft Dynamics 365 Business Central, part 1]( {{< ref "posts/Sali_Part1.md" >}})

In previous part we were talking about inter-application Dependencies and their layering which helps with correct dependency structure. But still we have a problem how to decide, which new requested feature will
go into which application to keep the architecture clean. To help with this decision making, we created few rules regarding this.

## SaLi Naming rule

> Is there a unique and concise name for the app that describes the functionality? If not, it’s a signal that you may be merging multiple "apps" scopes into one.

{{< fancybox path="/assets/SaLi" file="ArchitectNames.jpeg" caption="Architect thinking about different names for the app" gallery="SaLi 2" >}}

This is first rule, which helps with correct naming of the apps you are creating. Because name of app is defining scope of the app. Creating correct name means defining correct scope. If I have clear and simple scope, I can later decide correctly if requested functionality match the scope or not (see **Scoping** rule). This helps with fast and correct architecture decision process.

If I cannot create some simple and clear name for the app, may be the scope I am trying to name is too complex or wide. Of course, this could be solved by using some "generic" naming like "Per tenant app" or "Small modifications" or "All what you want changes", but all depends on your sense and scale of splitting your solution into more apps ([Split or not to split]( {{< ref "posts/PTEArchitecture1.md" >}})). This rule is not solving this "Split or not to split" problem, it just emphasize synchronicity between app Name and app Scope.

## SaLi Scoping rule

> If you are adding new thing into app (function, object...), ask "Is it in line with the naming of the app (function, object...)?" If not, put it somewhere else or rename the app.

{{< fancybox path="/assets/SaLi" file="AppScopes.jpeg" caption="Where I will put this change?" gallery="SaLi 2" >}}

This rule is similar to the **Naming** rule, but other way around - app already exists and you are deciding if the new feature match the scope of the app or not. Target is to not put functionality into app if it is not in line with the scope of the app - it means keeping the app "clean" - containing only functionality which are in line with the purpose and name of the app.

If I have app "External CRM integration", there must be only functionality fullfilling the generic External CRM Integration functionality, not e.g. some specific external CRM system requirements (these will be e.g. in separate app "SomeCRM Communication Interface"). As a result, some original functional requirements could be implemented as multiple functionalities added into multiple applications, because they requires changes in multiple scopes. At the end, you will need to split the requirement into multiple change requests for multiple apps (multiple responisibilities). This could be seen as complication, but we think that it is just helping with decomposition of the problem to smaller pieces which are mostly loosly-coupled (see **Decoupling** rule).

## SaLi Sharing rule

> When you add field (action/process) to a process/in-out app, think, ”Will you need it elsewhere when you uninstall that app?” if so, don’t put that into this app, but into some app in SHARED/DISCRETE layer.

{{< fancybox path="/assets/SaLi" file="SharingData.jpeg" caption="Sharing data between applications" gallery="SaLi 2" >}}

Placing data/processes/actions into correct layer (app) is helping with the whole architecture. E.g. if you place the field with some data into app in "Input/Output" layer, no othe app will have access to it, because othe apps cannot have dependency on "Input/Output" layer apps. Sometime it is correct (value specific for this specific app which is only for internal purpose) and this rule is trying to help us decide, if it is true in specific case or if this specific data/process will be "shared" with other apps. If yes, we need to push this change into apps in lower layers to be able to share it easilly. Do not forget, that lower layers are for "generic rules, business processes etc.", higher layers are for "specific things". This rule helps with implementing the generic things first and then implement their specific implementations.

Part of this decision is even decoupling generic data/processes from specific data/processes (connected to specific external system, internal app, etc.). E.g. field "Exported (count)" on sales document (which drives some internal workflow processes) will be generic field in some lower layer app, but field "Exported to SomeCRM (count)" will be probably in "Input/Output" layer app for "SomeCRM" integration only and will not be connected to generic workflow processes (because these processes will not work when you switch from "SomeCRM" to "AnotherCRM" integration - it means when you uninstall "SomeCRM" integration app).

Thinking about "What will happen when I remove this app" helps you to prepare the solution to situations when you decide to change part of the system. If you think about this, solution will be prepared for future changes (this is part of the SOLID priciples).

## SaLi Decoupling rule

> If your functionality is expected to work without the app on which it depends, then don’t make that dependency (and do it with help of some library or connector app, split the app...).

{{< fancybox path="/assets/SaLi" file="UninstallApp.jpeg" caption="What happen when I uninstall this?" gallery="SaLi 2" >}}

Rule is similar with the **Sharing** rule, but from another point of view. Thinking about "what happen when I remove ths app" helps with identification of tightly-coupled apps and validating if this relation is correct. Making all apps tightly-coupled means we have rigid system, which is hard to change (and it means against SOLID architecture). To make the system flexible means, we need to introduce more abstract apps (helping us to split the generic functionality from specific implementation) or implement different "bridge apps" inter-connecting multiple loosly-coupled apps. In case we need to remove/exchange one app, we just reimplement the bridge app, which is tightly-coupled with it. All this helps us to implement correctly the SOLID principles and unpair the generic things from their specific implementations.

And do not forget, "never say never" (we all know the customer's "*it will never happen!*").

## Summary

All the rules are trying to focus on the application level and dependency connected decisions. But in reality, you can apply them even on lower levels like objects or functions etc. You can try to specify your own questions which will help you to decide consistently in repeating situations. Creating such a questions/rules will help whole team to be in line with the goal and the architecture will by more consistent even when architect will be someone else.

All the rules are helping even with implementing all the SOLID principles, because they are trying to guide architect to define clear scopes and helps with separation of concerns (and connected SOLID principles). Do not forget, that architect doesn't need to be developer and doesn't need to understand technical language used by devs. Writing the rules in "common language" makes them understandable for everyone in the team. Naming all the things can help with the discussion about the architecture when needed.

Do not hesitate to contact me if you have some questions or comments!
