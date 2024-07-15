---
title: "Using Paket CLI for Business Central"
date: 2024-07-15T15:00:00+02:00
draft: false
categories:
- Business Central
- Architecture
- NuGet
- NVRAppDevOps
tags:
- Business Central
- NuGet
- Paket
- NVRAppDevOps
thumbnail: PaketLogo.png
images: 
    - src: assets/NuGet/PaketLogo.png
      alt: Paket Logo
      stretch: none
      removeBlur: false
---
In my previous article [NuGet-izing Business Central: Current Status](https://blog.kine.cz/posts/nugetizingbusinesscentralstatus/) I have mentioned [NVRAppDevOps](https://github.com/kine/NVRAppDevOps) powershell module we are using in our DevOps and that it supports using NuGet packages together with Microsoft Dynamics 365 Business Central. This article will be about how it works.

## Paket CLI

When we were starting to use NuGet packages for our BC apps, I was using nuget.exe tool only. Later, when we had some additional requirements, I have found [Paket CLI](https://fsprojects.github.io/Paket/index.html) and learned that it is much easier to use for my purpose. Thus we migrated to this tool for the consuming part of our pipeline (the process, when we need to download the dependencies to be able to compile our app or deploy it to new contianer etc.). It is much easier to use than nuget.exe, because it is not so tightly coupled with Visual Studio project like nuget.exe is.

And by using existing tool, developed by and for community which is much bigger and "older" (in using NuGet) than BC community, I can use the know-how already encoded into this tool for free and right now. Downloading the NuGet packages is not so simple as it can look on the first sight. Why to reinvent the wheel?

### How Paket works

### Paket.dependencies file

To use Paket, you need only the paket.exe and one text file with name "paket.dependencies". This file is describing all what you need to be able to download the dependencies. Description of the file is documented [here](https://fsprojects.github.io/Paket/dependencies-file.html).

Example could look like this:

```txt
strategy: min           # use the minimum version of transitive dependencies
lowest_matching: true   # use the lowest matching version of a direct dependency

source https://myaccount.pkgs.visualstudio.com/_packaging/myfeedname/nuget/v3/index.json username:"" password:"%PAT%" authmethod:basic

nuget NAVERTICAas.NaverticaMeters.71971f57-xxxx-xxxx-xxxx-5e0886784631 >= 21.0.0.0
nuget Microsoft.Application  >= 23.3.0.0  strategy:min, lowest_matching:true
nuget Microsoft.Platform  >= 23.0.0.0  strategy:min, lowest_matching:true
```

Source in our case is some Azure DevOps artifact feed. The URL could be found on the "Connect to Feed" info on your feed in Azure DevOps. After you click on this button, just select that you want to connect to NuGet e.g. frm Visual Studio and on the opened page you will see the Source where the URL is ready to copy. When using private feed, you need to set even the authentication mode to Basic and set password to some PAT. You can use system environment variable to get the value to not include it directly in your file, because the paket.dependencies could be commited as part of your source code. Just set the PAT environment variable to the PAT value and you will be able to use this source to download your packags.

*Strategy* and *lowest_matching* are options which sets version policy when resolving the dependency tree (if you want min or max versions for transitive dependencies and direct dependencies). Do not forget that you should use both policies in your pipelines, because both are valid and are testing or giving you different aspect of your application (see [this Areopa webinar](https://youtu.be/u-XCDwS4Vw0?si=YyaaKfOO_CcS5KHY) for more info).

Last lines are telling Paket which dependencies we want to download. Paket will automatically resolve whole dependency tree and versions of all the dependencies based on the requirements you set here. Paket is able to work even with other packages than NuGet, but we will be using only the NuGet packages for now.

You can see that it is not hard to generate this file based on the app.json of your app and we need only somehow add the sources. How we will generate the package names are given by the Rules we have defined. The rule is simple:

```txt
- pattern: <Publisher>.<AppName>.<Tag>.<AppId>
- max length: 100 chars. If longer, cut <AppName> to fit
- allowed characters (except separator '.'): a-zA-Z0-9_\-
```

Tag could be localization tag when the app exists with same AppId in multiple "flawors" (mostly only MS Apps will use this) or it could be 'symbols' if the package container only symbols or 'runtime' if it is runtime package. Else it is not used at all.

### Paket.lock file

During usage of Paket, the Paket.lock file is created. This file describe which versions of dependencies were used last time you resolved them. If you include this file as part of your repository and you will use only "Paket restore" command (see next section), you will get same versions of the dependencies every time. This can help you to make the process consistent and free from side effect of using different versions of dependencies every time you resolve them.

### Paket commands

When running Paket, you need to choose which command you want to run:

```cmd
Paket <command>
```

<command> is one from this list:

- Install - Compute dependency graph, download dependencies and update the folder. Will resolve new packages in the paket.dependencies and use already used versions for already resolved dependencies. Only new packages are added into paket.lock file, if exists.
- Update - Update dependencies to their latest version - use this when you want update the dependencies. Process will resolve the versions again and will store the new versions into paket.lock file. You can update only selected package if you want.
- Restore - will donwload the computed dependency graph as described by the paket.lock file. It means it will use same versions as when the paket.lock file was updated last time. No new versions will be resolved or updated.
- clear-cache - could be used to clear the local cache, e.g. when you unlist some packages on the source etc.

and there are other commands, but we do not need them right now.

### Workflow with Paket

- create paket.dependencies based on the app.json and add the source for the feed you want to use
- run "Paket install" to resolve the dependency graph when there are new dependencies in the file
- run "Paket update" when you want to update version of some/all dependencies and download them
- run "Paket restore" when you want to download same versions of dependencies as described by paket.lock file (restore the last state)

## NVRAppDevOps and Paket

### Usage from VSCode

In Powershell module NVRAppDevOps, there is cmdlet "Invoke-PaketForAL" which pack all the functionality into one command. You can use it like this:

```cmd
Invoke-PaketForAL -Sources 'https://myaccount.pkgs.visualstudio.com/_packaging/myfeedname/nuget/v3/index.json username:"" password:"%PAT%" authmethod:basic'
```

This is good e.g. for usage from VSCode, when you need to download AL symbols. Instead downloading symbols from some existing environment, you can download them as NuGet packages from your source feed.

If you run the command in same folder as app.json of your app is and the module will be able to run Paket.exe from default path, it should convert the info from app.json and create the paket.dependencies file for you, add the one source and will call Install paket command.

The internal flow of this cmdlet is this:

1. If *paket.dependencies* exists, read the content
1. If no -Sources are passed as parameter, use the Sources already existing in the *paket.dependencies* file
1. Update the *paket.dependencies* file with actual dependencies from app.json
    - When creating NuGet package ID for Business Central application, first look through sources for package with the AppId in name. If Found, use this package name
    - If no package with the AppId found on the sources, generate the NuGet package id based on the rules
    - Add the dependency into the paket.dependencies file with the version limits from *app.json*
1. Run the "Paket install" command. if system environment with name *PaketExePath* is set or the path is passed through *PaketExePath* parameter, use the value as path for the *Paket.exe*.
1. If *UsePackagesAsCache* is specified, the folder Packages is added into *al.packageCachePath* setting in *.vscode/settings.json* file. ALC then will use the .app files inside this folder automatically as it is using *.alpackages* by default.

It is good to add the *Packages* folder into *.gitignore* file, because it should not be part of the repository. Only the *paket.dependencies* and *paket.lock* should be commited into the repository.

Later you can use only simpler version without source parameter, if the paket.dependecies already exists:

```cmd
Invoke-PaketForAL 
```

This will reuse existing *paket.dependencies* file with sources and will just regenerate the dependecies package IDs and version limits. Then it will issue the "paket install" command.

### Usag from DevOps CI/CD pipeline

Same cmdlet from the NVRAppDevOps module could be used to download the .app files as part of your CI/CD. In this case, you will mostly use the "restore" command to restore the dependencies to state described in the paket.lock file. Or you can do other commands in connection with different *policy* parameter to download minimum or maximum versions of the dependencies.

You will probabbly reuse the sources already defined in the paket.dependencies file. You just need to set the environment variable used e.g. for defining password (when using e.g. %PAT% value as password) to authenticate correctly.

After NuGet packages are downloaded, you can use the package folder as source of symbols for the *alc* to compile your app, or as source of apps to deploy to target BC environment (probabbly skipping the MS apps because they will be already in the environment).

## Summary

I have created the Invoke-PaketForAl cmdlet to make the process of using NuGet packages in connection with Microsoft Dynamics 365 Business Central as easy as possible. Still, there is enough parameter to customize the result. I hope that i twill be the starting point to use the NuGet packages with BC instead downlaoding the symbols from existing environment. But still, we need at least the Microsoft NuGet feed for MS apps to have simple experience. See the ["NuGet-izing Business Central: Current status"]({{< ref "posts/NuGetizingBusinessCentralStatus.md" >}}) article about current status of the whole story. It could take some time, but I am ready for the future... ;-) Whole process is compatible with standard usage of the *Paket CLI* and is just automatizing some aspects of this. It means you can use *Paket* directly if you need some specific features which are not covered by the cmdlet. Creating of *paket.dependencies* fiel could be called as separate cmdlet *ConvertTo-PaketDependencies* if you need like this:

```powershell
ConvertTo-PaketDependencies -ProjectPath $ProjectPath -NuGetSources $Sources -Policy $Policy -MaxApplicationVersion $MaxApplicationVersion -MaxPlatformVersion $MaxPlatformVersion
```

In case of some problems or requests, do not hesitate to creaet issue on the **[NVRAppDevOps GitHub repo](https://github.com/kine/NVRAppDevOps/issues)**.
