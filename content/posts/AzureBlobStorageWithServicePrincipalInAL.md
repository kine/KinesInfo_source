---
title: "Accessing Azure Blob Storage with Service Principal from Business Central AL"
date: 2024-08-26T14:00:00+02:00
draft: false
categories:
- Business Central
- AL
- Azure
tags:
- Business Central
- AL
- Azure
mermaid: false
---

## Azure Blob Storage from AL

After long time I needed to solve some AL problem for my colleagues and I was not able to find the solution on internet and even from other MVPs. Today I had some time to dig into the problem and I have solved it.
I want to share the solution with you. But first, we need some introduction into the problem.

It is common to use Azure Blob Storage to store data and work with them from Dynamics 365 Business Central. Developers can use system app codeunit *"ABS Blob Client"* to call different methods of the storage. But this library implements only **Shared Key** (mostly used because it is the simples one) and **SAS** authorization. But the most recommended method is to use **Microsoft Entra ID with managed identities** to authorize the calls. And because our customers are taking the security seriously, we wanted to change the code from **Shared Key** auth to **Service Principal** (managed identity). I started to search for some example for AL regarding this, but very example is using just Shared Key auth.

Ok, time to do own research and learn something new. During the search I found some example in Python and based on this example it looked like easy task. When I connected the example with my knowledge about OAuth, the solution started to crystalize.

## Managed Identity auth for Storage Service

During analysis of the *"Azure Storage Service"* implementation in System application I hit the interface *"Storage Service Authorization"* which is used to do the authorization of the API calls. This interface defines only one procedure which needs to be implemented:

```AL
procedure Authorize(var HttpRequest: HttpRequestMessage; StorageAccount: Text);
```

When I looked on existing implementations, this method needs to set the HTTP Attributes of the request as needed to authorize the request. In our case, we need to set the *Authorization* header to *"Bearer token"*, where the Token is OAuth token for our Managed Identity for accessing the resource we need to access. Based on the examples I found it is just standard authorize OAuth call with client credentials where:

- ClientID is the ID of the managed identity (Entra ID app we registered for this purpose)
- ClientSecret is the secret for he managed identity
- AuthorizationURL is e.g. `https://login.microsoftonline.com/tenantid/oauth2/authorize/`
- RedirectURL could be empty (we are not using delegation)
- ResourceURL is `https://<accountname>.blob.core.windows.net/`

Thanks to codeunit OAuth2 it is easy to do this call by using this code:

```AL
if not OAuth2.AcquireTokenWithClientCredentials(ClientId, ClientSecret, AuthURL, RedirectURL, ResourceUrl, Token) then
    Error(TokenNotAcquiredErr);
exit(Token);
```

Then we can create own implementation of the *"Storage Service Athorization"* interface:

```AL
codeunit 50100 ServicePrincipalAuth implements "Storage Service Authorization"
{
    var
        Token: SecretText;
        ClientId: Text;
        ClientSecret: SecretText;
        AuthURL: Text;
        RedirectUrl: Text;
        ResourceUrl: Text;
        TokenNotAcquiredErr: label 'Failed to acquire token';
​
    procedure Authorize(var HttpRequest: HttpRequestMessage; StorageAccount: Text)
    var
        Headers: HttpHeaders;
        AuthToken: SecretText;
    begin
        if Token.IsEmpty() then
            Token := GetToken(ClientId, ClientSecret, AuthURL, RedirectUrl, ResourceUrl);
        HttpRequest.GetHeaders(Headers);
        if Headers.Contains('Authorization') then
            Headers.Remove('Authorization');
        AuthToken := SecretStrSubstNo('Bearer %1', Token);
        Headers.Add('Authorization', AuthToken);
    end;
​
    procedure SetPrincipalData(_ClientId: Text; _ClientSecret: SecretText; _AuthURL: Text; _RedirectURL: Text; _ResourceUrl: Text);
    begin
        ClientId := _ClientId;
        ClientSecret := _ClientSecret;
        AuthURL := _AuthURL;
        RedirectUrl := _RedirectURL;
        ResourceUrl := _ResourceUrl;
    end;
​
    local procedure GetToken(ClientId: Text; ClientSecret: SecretText; AuthURL: Text; RedirectUrl: Text; ResourceUrl: Text): SecretText
    var
        OAuth2: Codeunit OAuth2;
    begin
        if not OAuth2.AcquireTokenWithClientCredentials(ClientId, ClientSecret, AuthURL, RedirectURL, ResourceUrl, Token) then
            Error(TokenNotAcquiredErr);
        exit(Token);
    end;
}
```

When we have this implementation, the code for e.g. listing the Blobs is just few lines of code:

```AL
ServicePrincipalAuth.SetPrincipalData(ClientId, ClientSecret, AuthURL, RedirectURL, ResourceUrl);
ABSBlobClient.Initialize(AccountName, ContainerName, ServicePrincipalAuth);
ABSOperationResponse := ABSBlobClient.ListBlobs(ABSContainerContentTemp);
```

This implementation is not handling expiration of the token, it is bare minimum to do the call at least once.
