---
title: "How to publish your local BC containers to internet using IIS"
date: 2022-08-18T12:00:00+01:00
draft: false
categories:
- Business Central
- Infrastructure
tags:
- Business Central
- Infrastructure
- IIS
thumbnail: ProxyDesign.png
images: 
    - src: images/ProxyDesign.png
      alt: ProxyDesign
      stretch: none
      removeBlur: true
---

If you are doing Dynamics 365 Business Central development on your local (not Azure) containers, may be you want to have access to them from outside your local network without using VPN. And not only to BC Web Client, but even to API/OData and development endpoint. And of course you want to have them published with some trusted certificate to be able to use all the functionality like Business Central application, connection from Power BI connectors etc. It is handy when you are working with external workers like freelancers, because you do not need to setup VPN etc. for them. When using AAD authentication, just invite them as guest users to your tenant and you are ready to give them access as needed.

Different tools exists for solving that like [Traefik](https://traefik.io).

But if you want just easy solution using Windows IIS, you can use the Application Request Routing (ARR) feature of IIS for that.

## How it works

When we create the proxy server (server with Windows and IIS installed) and we put it somewhere into our DMZ (zone which is accessible from internet and can access local network), we can setup DNS server to send our requests for some specified subdomain with our "BC Environments" to that proxy. The proxy takes the request and will forward it to correct internal server/container and will transfer the response back to the sender. In my case, all our internal development environments are running as part of our dev. subdomain, thus I am able to forward everything going to *xxx.dev.ourdomain.com* to our local *xxx.dev.ourdomain.com*.

{{< fancybox path="/assets/Proxy" file="ProxyDesign.png" caption="Proxy architecture" gallery="Proxy" >}}

And if I set the proxy to listen on port 443, 7047, 7048, 7049 and 7085, I will make the web client, SOAP, OData/API, Development and Snapshot debugging available for people from outside.

## Needed components

### External DNS Entry

On your external DNS for your dev.ourdomain.com zone, you need to create wildcard entry of type CNAME leading to the external interface of your PROXY. It will translate anything in format xxx.dev.ourdomain.com to the proxy server address.

### Internal DNS Entries

On your internal DNS you need to translate same address as you want to use from outside, to local address of the server/container. It means, on your internal network the address xxx.dev.ourdomain.com should be translated to the local address of the server/container. In this way we do not need any "translation" table which will transform the external name to internal one and we can just maintain the local DNS entries. Having same external and internal address is even good thing when you want to use AAD authentication in BC.

### Forward Proxy

On the proxy, install IIS feature with ARR and URL Rewrite. You can use [this documentation](https://docs.microsoft.com/en-us/iis/extensions/configuring-application-request-routing-arr/creating-a-forward-proxy-using-application-request-routing) to install and setup what is needed. Enable the SSL offloading to be able to have different internal and external SSL certificate. When the offloading is enabled, the proxy will act as man-in-the-middle. It means the external client is communicating with the external https endpoint of the proxy, which have some externally trusted certificate installed on it. The proxy then takes the request and will create new connection to the local resource and send the request there. The local resource could have internally trusted certificate, or could use http protocol without certificate if you want.

Set the rules like this:

- Global forward rule
{{< fancybox path="/assets/Proxy" file="ProxyRule1.png" caption="Proxy rule 1" gallery="Proxy" >}}
- Rule to handle OAuth request correctly (without this the OAuth will end on the container instead the Microsoft site)
{{< fancybox path="/assets/Proxy" file="ProxyRule2.png" caption="Proxy rule 2" gallery="Proxy" >}}
- Rule to handle OAuth request correctly (without this the OAuth will end on the container instead the Microsoft site)
{{< fancybox path="/assets/Proxy" file="ProxyRule3.png" caption="Proxy rule 3" gallery="Proxy" >}}

Set the default web site on the IIS to listen on the required ports like 443, 7048, 7049 and 7085. Open these ports for access from internet on the firewall.

## Conclusion

If everything is working correctly, you should be able:

- connect to the xxx.ourdomain.com web client and other endpoints from local network
- connect to the xxx.ourdomain.com web client and other endpoints from internet without using VPN
- have trusted certificate placed only on the proxy server on the default site
- have internal certificate on your environments (proxy server must trust them)

If you are missing some info here, let me know and I will update the article as needed.

Warning! This setup is forwarding everything going to the *.dev.ourdomain.com to the proxy, thus it could be used for attack to your internal infrastructure, if you have more services available than just the BC containers on your internal dev.ourdomain.com domain. You need to be aware of that and you can somehow limit this by modifying the rules e.g. to forward only request having format of the names of your BC environments etc. Best is to use subdomain where is nothing else than the environments.

Enjoy work with the environments withouth VPN!
