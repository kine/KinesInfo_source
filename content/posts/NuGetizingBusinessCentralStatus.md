---
title: "NuGet-izing Business Central: Current status"
date: 2024-07-12T10:00:00+02:00
draft: false
categories:
- Business Central
- Architecture
- NuGet
tags:
- Business Central
- NuGet
thumbnail: NuGetDelivery.jpeg
images: 
    - src: assets/NuGet/NuGetDelivery.jpeg
      alt: NuGet delivery
      stretch: none
      removeBlur: false
---
In my last sessions on different conferences and webcasts I was talking about my dream regarding using NuGet technology to distribute and consume Business Central .app files. These sessions were starter point for make the whole dream reality. But we are not there yet. We just started the trip. This blog article want to summarize current status of this trip.

## Requirements

To make the dream reality, we need multiple things to happen:

1. Define rules for creating the NuGet packages in connection with Microsoft Dynamics 365 Business Central - status: **DONE!** (see e.g. [my and Freddy's BCTechDays session "When a dream comes true"](https://youtu.be/JpmPqDM-hzU?si=FHoHB8qP4bpACzje))
1. Microsoft must publish their apps as NuGet packages on some public feed - status: **Finished** (updated 14.10.2024) - [MS Apps public feed](https://dynamicssmb2.visualstudio.com/DynamicsBCPublicFeeds/_artifacts/feed/MSApps), [MS Apps Symbols feed](https://dynamicssmb2.visualstudio.com/DynamicsBCPublicFeeds/_artifacts/feed/MSSymbols)
1. (optional) Microsoft need to publish AppSource apps symbols as public feed - status: **Finished** (updated 14.10.2024) - [public Artifact feed](https://dynamicssmb2.visualstudio.com/DynamicsBCPublicFeeds/_artifacts/feed/AppSourceSymbols) - updated when publisher publish new version of some app
1. Partners need to publish their apps as NuGet packages on some feeds following the rules - status: **Depends on partner** (unknown)
1. Partners need to be able to consume NuGet packages following the rules during CI/CD and development - status: **Depends on used technology (see next chapter)**

## Adoption of NuGetizing for BC in DevOps solutions

One part of the NuGetizing the BC is possibility to consume and produce the NuGet packages during your CI/CD pipelines.

Here is the list of existing managed solutions and tools for DevOps for Business Central and their status of NuGet support:

- [Alpaca](https://www.cosmoconsult.com/cosmo-alpaca/): On the roadmap (WIP)
- [ALOps](https://alops.be/): On the roadmap (WIP)
- [BCCH (BCContainerHelper)](https://github.com/microsoft/navcontainerhelper): WIP
- [AL-Go for GitHub](https://github.com/microsoft/AL-Go): WIP (updated 30.9.2024)
- [NVRAppDevOps](https://github.com/kine/NVRAppDevOps) (powershell module): Done! (since version 2.8.4-beta20, use the Invoke-PaketForAl)

If you know any other DevOps solution for Business Central, let me know and I will add it here.

## Adoption of NuGetizing for BC in VSCode

Another aspect of NuGetizing Business Central is possibility to consume NuGet packages from VSCode to replace the "AL: Download Symbols" action.

Here is list of tools which could help with this (updated on 14.10.2024):

- [NVRAppDevOps](https://github.com/kine/NVRAppDevOps) (powershell module): Done! (since version v2.8.4-beta20, use the Invoke-PaketForAl)
- [AL NuGet Helper](https://marketplace.visualstudio.com/items?itemName=PatrickSchiefer.al-nuget-helper) from Patrick Schiefer (VS Code extension): Released first version 10.10.2024, WIP, inside using Paket CLI

## Summary

You can see, that tools are popping up, but without the feed infrastructure from Microsoft and other partners, it is hard to use them. In our company we are using NuGet packages whole time we created our CI/CD pipelines for Business Central. Thus we have even our internal fake packages for MS Applications and we are wrapping the 3rd party .app files as NuGet packages for our internal needs. But it is just complication and it is not part of the dream (it is rather nightmare). When the feed infrastructure from MS will be finished, it will be much easier to start using the tools to incorporate NuGet packages into your processes.

**Regarding consuming the NuGet packages** - if you are early adopter kind of person, you can start now and solve these things yourselfs. Rest needs to wait till the MS feeds are finished at least.

**Regarding producing NuGetPackages** - you can start now, because at least two tools are already there for you - BCCH and NVRAppDevOps cmdlets (New-BcNuGetPackage in BCCH and New-ALNuSpecForAppFile, New-ALNuSpec and New-ALNugetPackage in NVRAppDevOps cmdlets). And even when you do not want to use these tools, it is easy to generate the XML .nuspec file and create the package by using current tools like NuGet.exe or Paket.exe CLI. There is no hidden magic. Just follow the rules for naming the packages. You do not need to consume the packages yet, but you can build your feeds in this way and be prepared later, when we will be closer to the goal.
