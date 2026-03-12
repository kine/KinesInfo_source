---
title: "Business Central Development Serie - Part 2: Using AI for BC development"
description: ""
date: 2026-03-11T00:00:00+01:00
draft: false
author: "Kamil Sacek"
categories:
- Business Central
- Architecture
- Development
tags:
- Business Central
- Architecture
- Development
- Best Practices
- AL
- PTE
- AppSource
mermaid: false
thumbnail: BCDevSeriePart02.svg
images:
    - src: assets/BCDevSerie/BCDevSeriePart02.svg
      alt: "Business Central Development Serie - Part 2: Using AI for BC Development"
      stretch: none
      removeBlur: false
---
Welcome to the second (or first? May be error by 1...) part of the Business Central Development Serie! In the introduction (aka part 1) I explained why I started this serie and what I want to achieve with it. I also shared the plan of topics which I want to cover. In this second part I will start with the first topic from the plan - Using AI for BC development.

AI tools are becoming more and more popular among developers. They can help you to write code faster, to find bugs, to learn new things. But they are not magic. They are tools which can be very useful if you know how to use them, but they can also lead you to problems if you do not understand their limitations and how to get the best results from them.

I am beginner in this AI world and I see much better experts in vibe coding extensions for Business Central. But I already see the gain in productivity and what I am able to deliver thanks to AI. But I also realise that using AI for write code is just a small piece of the puzzle.

## AI is good servant but bad master

As always, everything can be used for good or for bad. In good way or wrong way. AI tools are no exception. They can be very helpful if you use them as a tool to assist you in your work, but they can also lead you to problems if you rely on them too much or if you do not understand their limitations.

Some people thing that they can use AI to create the solution for them easily and they do not need the expansive experts anymore. They just need to ask AI to write the code and they will have the solution. But it is not that simple. AI can write code for you, but as always, it will do only what you ask it for. And from my experience, if you want to make someone angry, "do exactly what he ask you to do". If you do not know what to ask, if you do not understand the problem, if you do not understand the solution, if you do not understand the consequences of the solution, you will get a code which is working, but it may not be the right solution for your problem. It may be a solution which is not maintainable, which is not performant, which is not secure, which is not following best practices. And then you will have problems in the future when you need to maintain it, when you need to fix bugs, when you need to add new features, when Microsoft change something in the platform/standard code. Do not trust the hype you can see on internet. Yes, it works, it works best for Proof of Concept, fast prototyping, learning, but it is not a replacement for understanding the problem and the solution. For using it for production code, you need to understand what you are doing, what you are asking for and what you are getting.

## Corner stone - guidelines, rules, best practices

Before you start using AI for writing code, you need to have some guidelines, rules and best practices in place. You need to know what are the things which are important for your solution, what are the things which are not important, what are the things which are must have and what are the things which are nice to have. You need to have some standards for your code, for your architecture, for your testing, for your performance. You need to have some rules for how to use AI, what to ask for, how to review the code, how to test the code. If you have some internal documentation how to write the code, you can use it as a base for your AI instructions. If you do not have it, there are already some tools or repositories which can help you.

Part of the instructions is even workflow for the agent. For example, if you are using Azure DevOps Work items to describe user stories etc., it is good to describe how the agent should work with the Work Items.

In general, if you are ready for new developers in your team in a way, that everything they need to know for their work is described in some documentation, then you are ready to use AI for writing code. Because you can use the same documentation as a base for your instructions for AI. If you do not have such documentation, you need to create it first or get it from someone else (there are public repositories and tools which can help you).

Just take the AI Agent as your virtual team member. You need to onboard it, you need to give it the guidelines, rules and best practices, you need to review its work, you need to test its work. It is not a magic tool which will do everything for you without any input from you. It is a tool which can help you if you know how to use it.

Your starting point could be [this site](https://alguidelines.dev/docs/agentic-coding). It includes even list of community sources and tools you can use in your process.

## Extension Architecture in time of Agentic development

In my article [Architecture of PTE - Split or not to Split?]({{< ref "posts/PTEArchitecture1.md" >}}) I explained how to think about architecture of your customer solution (or even AppSource product). Is Agentic development changing something on this? In my opinion not much. It adds another dimension to the decission. If you are working on big monolitic app, agent can have problem with context size. If agent is working with smaller apps, it will be easier to understand the context. But in monolith you do not have problems with implementing more complex requests (which are changing multiple areas of the app). In case of multiple depending apps you need to handle the splitting of the requirement to separate changes in separate applications. It means it is more comlpex to use e.g. cloud GitHub Copilot to develop the solutuion (leading to multiple depending Pull Requests). This is general problem of using Agentic development in MultiRoot workspaces. But not impossible to solve and the PROs (easier maintenance, smaller apps to deploy, single responsibility principe) can still overweight the CONs. It is just something to consider when you are thinking about your architecture and your development process.

## Developing without CI/CD solution is not a good idea

If you want to automatize your code writing process, you first need to automatize your build and delivery process. If you do not have CI/CD solution in place, you will have problems with testing and deploying the code which is generated by AI. And in case something goes wrong. And do not forget - you are still responsible for the code you deliver to the customer. Thus you need to have way how to control what your virtual colleague is producing.

There are multiple choices:

### Platform

You can choose to use Azure DevOps or GitHub. Both of them have good integration with Business Central and both of them have good tools for CI/CD. It is just a matter of your preference and your existing tools. Just do not forget that there is GitHub Copilot, not Azure DevOps Copilot. But still you can use Azure DevOps to manage Work Items, but have code repositories on GitHub. Then you can have your Pipelines on Azure DevOps or you can use GitHub Actions. It is up to you. Or just use the GitHub for everything. All depends on your current situation and your preferences.

### Tools

For Azure DevOps you can use commercial tools like [ALOps](https://www.alops.be/) and [NAVBaaS](https://robberse-it-services.nl/navbaas) or you can use open source tools like my [NaverticAL Build and Release Tasks for Business Central](https://marketplace.visualstudio.com/items?itemName=Kine.naverticaltasks). For GitHub you can use Microsoft [AL-Go for GitHub](https://github.com/microsoft/AL-Go). And then there are mich wider tools, solving not only CI/CD but also other parts of the development process, like [COSMO Alpaca](https://github.com/marketplace/cosmo-alpaca) and is supporting both platforms.

## Agentic vs Manual development

To be able to correctly use agentic development, you need to set your KPIs and your goals. Because sometime managers tends to prioritize "productivity" over "quality" ("oh, look, I am able to deliver this functionality in day instead week"). But the main focus shoul be **technical debt, performance, maintainability and delivered value to costumer**. Agentic development helps you to deliver **richer solution**, because it is just few minutes to produce things which were mostly skipped by developers because time/cost pressure, like **automatic tests, technical documentation, user documentation, UAT scripts**, etc. But it can also lead you to deliver more code which is not following best practices, which is not maintainable, which is not performant. Sometime developers are loosing time by trying explain the AI Agent what to fix instead writing two lines of code manually. Sometime it is just faster to write the code manually than to explain the AI what you want.

It is clear that companies which will not use Agentic development will be less competitive on the market. But it is also clear that companies which will use Agentic development without understanding it, without setting the right guidelines, rules and best practices, without setting the right KPIs and goals, without having the right architecture and CI/CD solution in place, will have problems with maintainability, performance and technical debt in the future. It is just a matter of how you are using it.

And do not forget that you still need to have human developers who understand the problem, understand the code, architecture. But "mind the gap" - if we will use AI Agents to write the code, how we will raise the human developers? If we remove the junior developers from the process by replacing them with AI Agents, how we will have the next generation of developers who will understand the code, architecture, best practices? Yes, it is similar problem like when calculator didn't replaced mathematicians, but it is still something to think about. We need to find the ways how to bridge this gap.

## Responsibility

And the last but not least - responsibility. You are still responsible for the code you deliver to the customer. You are still responsible for the performance, maintainability, security, etc. of the code you deliver. You are still responsible for the bugs you deliver to the customer. You are still responsible for the technical debt you deliver to the customer. You are still responsible for the value you deliver to the customer. You can blame AI, but human is the one responsible. Using AI will not remove your responsibility. It could make it just worse by **making much bigger damage in much shorter time**. So be careful how you are using it!

## Summary

AI tools are becoming more and more popular among developers. They can help you to write code faster, to find bugs, to learn new things. But they are not magic. They are tools which can be very useful if you know how to use them, but they can also lead you to problems if you do not understand their limitations and how to get the best results from them. We all are learning how to use them, how to avoid the problems, but we know that the World is changing and we cannot undo the change. And the change is faster than before, and it will not slow down. But as a developers we know that "only constant in the World is change". We should be ready for it, we should adapt and we should use it to our advantage. But we should also be careful how we are using it, because it can lead us to problems if we do not understand it.
