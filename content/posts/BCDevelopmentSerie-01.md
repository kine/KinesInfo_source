---
title: "Business Central Development Serie - Part 1: Introduction"
description: "Introduction to blog serie about best practices and patterns for developing Microsoft Dynamics 365 Business Central extensions. Covering architecture, dependency management, testing, performance, delivery and maintenance."
date: 2026-03-05T00:00:00+01:00
draft: true
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
thumbnail: BCDevSerieIntro.svg
images:
    - src: assets/BCDevSerie/BCDevSerieIntro.svg
      alt: "Business Central Development Serie - best practices and patterns for BC extension development"
      stretch: none
      removeBlur: false
---
If you are reading this, may be you are one of the growing number of partners and developers who started to develop extensions for Microsoft Dynamics 365 Business Central. And may be you started thanks to the AI tools which are making the development much easier than it was before. But AI is not solving everything. It can write the code for you, but it will not tell you how to architect your solution, how to deliver it to customer, how to maintain it. And these are the things which are making the difference between "it works" and "it works and it will work next 20 years". Do not forget, that development for ERP/Business application is not only one time delivery, but it is a long term commitment to your customers. You need to be sure that your solution is not only working today, but it will be working and maintainable in the future.

I have almost 25 years of experience with development in NAV/Business Central world (and 40 years of experience with development at all) and I have seen many things going wrong. Not because developers were bad, but because nobody told them the rules, patterns and best practices. These things are scattered across different blogs, sessions, documentation and heads of experienced developers. I want to collect them into one place. This is what this blog serie is about.

## Why this serie?

With AI assisted development, barrier to start developing for Business Central is lower than ever. And it is good thing! More partners developing means more solutions for customers, more competition and more innovation. But it also means more developers who are not aware of things which are not written in the documentation. Things which you learn by doing, by failing, by discussing with others. Things which are making the difference between good and bad solution.

I am not saying that I know everything. I am sure that I do not. But I have some experience which I want to share and I hope that this serie will be starting point for discussions about best practices in our community. And I am open to learn from you too.

## What I will cover

In this serie I will try to cover topics which are important for developing and delivering solutions for Business Central. Here is the plan (which can change based on the feedback, discussion and my own learning during writing the serie):

1. **Introduction** - you are reading it right now
1. **Using AI for BC development** - how to use AI tools effectively for Business Central development, what they can and cannot do, how to get the best results and avoid common pitfalls
1. **Types of apps** - PTE vs AppSource app, what are the differences, what are the specific things you need to know for each type, when to choose which
1. **Architecture of extensions** - Split or not to split your solution into multiple applications? When, why, how? What are the PROs and CONs?
1. **Dependency management** - how to handle dependencies between your apps, how to use NuGet packages, how to avoid dependency hell
1. **Development process** - how to setup your development environment, how to use Git, how to do code reviews, how to use CI/CD
1. **Testing** - why and how to test your apps, what types of tests you should write
1. **Performance** - how to write performant code, what are the common performance mistakes, how to identify and fix performance bottlenecks
1. **Delivering to customer** - how to deliver your solution to customer, how to handle upgrades, how to handle data migrations
1. **Maintenance** - how to maintain your solution, how to handle bugs, how to handle new versions of Business Central

Some of these topics I have already touched in my previous articles (like [Architecture of PTE - Split or not to Split?]({{< ref "posts/PTEArchitecture1.md" >}}) or [SaLi architecture]({{< ref "posts/SaLi_Part1.md" >}})). In this serie I will reference them and may be extend them with new insights.

## Who is this for?

This serie is for everyone who is developing or planning to develop extensions for Business Central. It does not matter if you are experienced developer or if you just started. I will try to explain things from the basics, but I will also add some advanced topics for experienced developers.

But this serie is not only for developers. I will try to add information which could be useful for **consultants**, **project managers** and **decision makers** too. Because many decisions about the architecture, splitting the apps, choosing the right approach etc. are not purely technical decisions. They are business decisions which have technical consequences. And understanding the technical side will help you to make better decisions, ask right questions and communicate with developers more effectively. I will try to mark such sections clearly, so you can find them easily even if you do not want to read the whole technical details.

## One more thing

I want to emphasize one thing: **AI is a tool, not a replacement for understanding**. It is great that AI can help you to write code faster. But if you do not understand what the code does, why it is written in this way and what are the consequences, you will have problems sooner or later. Use AI to help you, but invest time to understand the things. It will pay off.

And do not forget - there is no silver bullet. Every solution, every customer, every team is different. What works for one may not work for another. But having the knowledge about best practices and patterns will help you to make better decisions in your specific situation.

I hope that you will find this serie useful and I am looking forward to your feedback and discussions!

Let's start the journey!
