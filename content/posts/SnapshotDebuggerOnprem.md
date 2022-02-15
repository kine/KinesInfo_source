---
title: "Snapshot debugger on OnPrem - what you need to know to set it up"
date: 2022-02-15T11:00:00+01:00
draft: false
categories:
- Business Central
- Administration
- AL
tags:
- Business Central
- AL
- Administration
- SSL
---

When you want to use Snapshot Debugging on your OnPrem environment, may be you can hit this error when trying to initialize it:

```
Error: The SSL connection could not be established, see inner exception.
...
```

In some cirumstances you can even have problem to start the BC Service when you enable the Snapshot Debugger endpoint.

## What is the problem?

For standard endpoint, which we are using for longer time already, like client endpoint (default port 7046), OData (7048) and Dev (7049), the Management console for BC Server is automatically setting some things in background when you save the settings in it. But for the Snapshot debugger, this is not done and you need to do it yourselfs.

## URLACL

First thing is, that for the user account, under which is BC Server running, usage of the HTTP address is reserved [(see this)](https://docs.microsoft.com/en-us/windows/win32/http/add-urlacl). Manually it could be done by running this command as admin on the server:

```cmd
netsh http add urlacl url=https://+:7083/BC/  user='NT AUTHORITY\NETWORK SERVICE'
```

You need to modify the URL if you are not using SSL (use http instead https) and port number/instance name. User needs to be the one under which the service is running.

To list existing urlacl you can use this command:

```cmd
netsh http show urlacl
```

If this is not done, you will ger some error about "unable to listen" on the specified address in your event log.

## SSLCERT

Second step is to assign certificate to the address, if you are using SSL for the endpoint.

To use SSL it is not enough to set the certificate in the BC Service Management console (in the BC Service configuration), but the certificate must be assigned to the address.

This is done again through netsh command:

```cmd
netsh http add sslcert certhash=<thumbprint> appid='{00112233-4455-6677-8899-AABBCCDDEEFF}' ipport=0.0.0.0:7083
```

Use the thumbrint of the certificate you want to use (e.g. same like in the BC Service configuration), appid could be any GUID (e.g. like in th example), and ipport must be the port for which you are assigning the certificate.

To list existing assignments you can use this command:

```cmd
netsh http show sslcert
```

If you want to assign certificate to port, where already is certificate assigned (e.g. when renewing expiring certificate), you need to delete the sslcert first:

```cmd
netsh http del sslcert ipport=0.0.0.0:7083
```

## Some hicups

Sometimes, when you are changing the SSL settings on the BC server, you can get into issues that when saving the configuration you get some errors that Certificate cannot be registered or something similar. It is mostly in situations, when the settings and the current state of the SSLCERT is not in line - it means, you are disabling SSL but SSLCERT is not registered (e.g. after you have copied configuration from another instance having SSL enabled and you current instance had it disabled) or vice versa. In this case you need to fix this discrepancy (when disabling SSL, create the SSLCERT assignment first, when enabling, remove the existing assignemnt first).
