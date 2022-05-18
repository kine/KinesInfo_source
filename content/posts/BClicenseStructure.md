---
title: "Using BCLicese file from powershell"
date: 2022-05-18T00:00:00+01:00
draft: false
categories:
- Business Central
- AL
- OnPrem
- PTE
- PowerShell
tags:
- Business Central
- AL
- OnPrem
- License
- PTE
- PowerShell
mermaid: false
featured_image: /LocalFS/Diagram.png
---

## New License format .bclicense

If you are working with OnPrem customers having Microsoft Dynamics 365 Business Central, you have already noticed the new license file format .bclicense. It was created by Microsoft to have modern way how to pass the licensing info without limits of the old .flf format (limited max size etc.). It had some side-effect (bugs) on beginning but I hope that they are solved (never tested yet). But today I am not writing about this file format because Customers, but because it could be handy for Partners/Developers and their CI/CD pipelines.

## Why developers should look at .bclicense file

Because you need to test that all your objects in your app are assigned in the Customer's license. Because world is not ideal, still there are plenty of customers using OnPrem for running their BC environments. And it means you need to assign object IDs into the license when you are developing some PTE. Forgetting to do so is one of the top reasons of problems during or after releasing new version of PTE into customer's environment.

One way is to use the customer's license during running your Automated Tests. You can do that easily by importing the customers license into your container/environment and run the tests. But if you want to fail fast if something is missing, and you want to be sure that everything is ok, it will not be enough. Waiting for the container to be created and the test finish will take time, and if there is not enough code covered by tests, you can still have problems. But it is still good to use the customer's license for the tests, because you can catch the missing object permissions e.g. when doing something with tables which are limited in access by default (ledger entries etc.).

## How to use .bclicense to check permissions

If you take look at .bclicense file, you will see that it have text header as old .flf file, and rest is XML. We are interested in the XML itself, because it keeps all the info we need to work with. If we will be able to read it and work with it as with xml document, we can go through the permissions and compare them e.g. with objects we have in our AL app. But how we can access the xml part of the file in powershell? It is not so hard. Just use this PowerShell script:

``` powershell
$filecontent = get-content -Path my.bclicense
[xml]$xml=($filecontent[$filecontent.count-1]).Remove(0,2)  
```

After that, we have the license as XML in our $xml variable. The expression will take last line from the file (the whole XML is one-liner at the end) and remove the beginning two characters (2x U+feff).

And now you can do just the standard XML tricks with the variable to check what you want. The structure looks like this:

``` xml
<License>
    <Properties>
        <Property name="xxx" type="yyy" id="nnn"><![CDATA[datadatadata]]></Property>
        ...
    </Properties>
    <PermissionCollections>
        <pl t="ObjectType" c="count">
            <ps>
                <p f="from" t="to" pbm="PermissionMask" />
                ...
            </ps>
        </pl>>
        ...
    </PermissionCollections>
    <Signature>
      ...digital signature of the file...
    </Signature>
</License>
```

As you can see, it is simple XML. We are interested in the node */License/PermissionCollections*. There we can find node for each license Object Type (TableDescription, TableData, System, MenuSuite, Codeunit, Page, FieldNo, Report, Query, XMLPort, Dataport, Form, LimitedUsageTable). In the *ps* (permission set?) node we can find the *p* node (permission?) for all the ranges in your license. It includes even ranges which you do not have in your license assigned, because such a ranges could have still some permissions in the license assigned, like "-MD-" permissions to be able to remove old unnecessary objects. 

### PermissionMask value meaning

Based on analysis of the file I found this bitmap mask for the PermissionMask value:

```
d mirX DMIR
```

Lowest bit of the value is on the right. It means the R permission is assigned if the value is odd and is not assigned if value is even.

Examples:

```
d mirX DMIR
------------
0 0000 1000 - ---D- - 8
0 0000 1100 - --MD- - 12
0 0000 1111 - RIMD- - 15
0 0001 0000 - ----X - 16
0 0001 0001 - R---X - 17
0 0001 0010 - -I--X - 18
0 0001 1000 - ---DX - 24
0 0001 1001 - R--DX - 25
0 0001 1010 - -I-DX - 26
0 0001 1101 - R-MDX - 29
0 0001 1111 - RIMDX - 31
0 1100 0001 - Rim-- - 193
0 1100 1001 - RimD- - 201
1 0100 0101 - RiMd- - 325
1 1000 0011 - RImd- - 387
1 1100 0001 - Rimd- - 449
1 1110 0000 - rimd- - 480
```

Thus you need to be aware that it is not enough to test if the ID of your object is included in some range, but you need to check if there is X permission to execute it on the range, M and I permission to be able to modify or create the object by publishing standard APP package (if these permissions are missing, you can only publish such an object by Runtime Package) etc.

You can test the permission e.g. by this function:
```powershell
function Check-PermissionMask
{
    param(
        [Parameter(Mandatory=$true,
                   ValueFromPipeline=$true,
                   ValueFromPipelineByPropertyName=$true,
                   HelpMessage="Permission value")]
        [int]$PermValue,
        [Parameter(Mandatory=$true,
                   ValueFromPipeline=$true,
                   ValueFromPipelineByPropertyName=$true,
                   HelpMessage="Permission to test")]
        [validateset('r','m','i','d','X','R','M','I','D')]
        [string]$PermToTest
    )

    $PermMask = 'RIMDXrimd'
    $TestValue = 1 -shl $PermMask.IndexOf($PermToTest)
    Return [bool]($PermValue -band $TestValue)
}

# Test of the function:
Check-PermissionMask -PermValue 15 -PermToTest 'R'
True
```

## CI/CD process

Thanks to the .bclicense you can extend your pipelines to test if license was correctly extended with new objects and you do not need to wait for the container (and fail as early as possible). You still need to solve one thing which is out of scope of this post - how to get the list of objects in your app without creating the container and publishing the app first. I am sure there will be some way (like going through all .al files and do some basic text parsing to find all the object headers, but it still could be complex e.g. to cover correctly temporary tables etc.). If you have some tip regarding that, do not hesitate to share it in comments. If I will create something regarding that, I will try to share it.

## Summary

You can use the new .bclicense format to work with the data inside the license easily. But you need to solve few things first, like where the actual license for the customer will be stored, how to get the list of objects in app etc. Use the knowledge as you need, I hope that this article will save you some time you will need to spend on analysis of the file yourself. I am sure that there will be plenty things you can thing out around this.

## P.S.: Using unsupported DLL included in BC installation

After publishing this article, I got info from Steffen Balslev from Microsoft, that there is DLL for handling the .bclicense file I can use to read the info. Of course, this dll and using it in this way is **unsupported** and **undocumented**, it means you are **using it on your own responsibility**. In reality, there are two DLLs - one for .netstandard (*Microsoft.Dynamics.BusinessCentral.Bcl.dll*) and one for .net framework (*Microsoft.Dynamics.BusinessCentral.BclFwk.dll*). I succeeded to use the .net framework version  from PowerShell to handle the license file. The .netstandard one is missing some reference (but it could be my fault). 

Here is my PowerShell code which loads the dll for you:

```PowerShell
#Download latest w1 artifact
$ArtifactPath = Download-Artifacts -artifactUrl (Get-BCArtifactUrl -type OnPrem -country w1 -select Latest) -includePlatform
#Find the dll
$DLLPath = Get-ChildItem -Path $ArtifactPath[1] -Filter 'Microsoft.Dynamics.BusinessCentral.BclFwk.dll' -Recurse -File

#region Load the assembly
$MSAzureKeyVaultPath = Get-ChildItem -Path $ArtifactPath[1] -Filter 'Microsoft.Azure.KeyVault.dll' -Recurse -File |Select-Object -First 1
$NewtonsoftJsonPath = Get-ChildItem -Path $ArtifactPath[1] -Filter 'Newtonsoft.Json.dll' -Recurse -File |Select-Object -First 1
$MSAzureKeyVault = [Reflection.Assembly]::LoadFile($MSAzureKeyVaultPath.FullName)
$NewtonsoftJson = [Reflection.Assembly]::LoadFile($NewtonsoftJsonPath.FullName)

#Some magic to resolve dependencies
$OnAssemblyResolve = [System.ResolveEventHandler] {
    param($sender, $e)
    foreach($a in [System.AppDomain]::CurrentDomain.GetAssemblies()) {
        if ($a.FullName -eq $e.Name) { return $a }
    }
    if ($e.Name -like 'Microsoft.Azure.KeyVault*') {return $MSAzureKeyVault}
    if ($e.Name -like 'Newtonsoft.Json*') {return $NewtonsoftJson}
    return $null
}
[System.AppDomain]::CurrentDomain.add_AssemblyResolve($OnAssemblyResolve)
try {
    Write-Host "Loading assembly"
    Add-Type -Path $DLLPath.FullName -Verbose
}catch { 
    Write-Host "Exception:" -ForegroundColor Red
    $_.Exception.LoaderExceptions
}

[System.AppDomain]::CurrentDomain.remove_AssemblyResolve($OnAssemblyResolve)
#endregion
```

The final use of the loaded assembly could be like this:

```PowerShell
$LicenseFile = 'my.bclicense'
$stream = [System.IO.StreamReader]::new($LicenseFile)
$reader = [Microsoft.Dynamics.BusinessCentral.License.BcLicense.LicenseReader]::new($stream.BaseStream)

$reader.GetObjectRangePermission("Page",18)

```

Output will be like this:

```PowerShell
RangeStart : 4
RangeEnd   : 56
Read       : Direct
Insert     : None
Modify     : Direct
Delete     : Direct
Execute    : Direct
Expiry     :
```

In this way, you do not need to parse the values etc. Everything is done by the class created for this purpose. But you are not in charge of the code. Choose wisely!