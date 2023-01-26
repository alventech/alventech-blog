---
title: Kerberos authentication and HTTP header size
author: Stein-Erik Alvestad
date: 2019-01-06T18:31:07+00:00
url: /kerberos-authentication-and-http-header-size/
categories:
  - Kerberos
  - on-prem AD

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: "mitchellluoFWoqldWlNQunsplash.jpg"

toc:
  auto: false

---

### Introduction

The last 4 years I have worked with developers to use modern Identity protocols like (SAML, OAuth, OIDC) on ADFS, Azure AD Enterprise Applications, Azure Application Proxy or G Suite for their applications.

But from time to time I come over applications that cannot use ADFS or Azure AD etc, and the last time happened just before Christmas when I was working with a customer who had problems with SSO.  

As usual, I was lacking documentation from the vendor on how the application worked. I got some information about LDAP binding service being used in the application and that the application did not prompt any errors. I was informed that the users were migrated from old AD to new AD and the application "App1" was re-established on new servers, but config was "identical".

I fired up Fiddler from a Windows pc and started tracing the request.  
I could see a trace of HTTP 400 error. This error Is related to the user Kerberos token size request header <https://support.microsoft.com/be-by/help/2020943/http-400-bad-request-request-header-too-long-response-to-http-request>  
I could see Kerberos authentication being used indicated by the YIIe negotiate and the Auth pane in Fiddler verified this as well from the screenshot below.

![image](/wp-content/uploads/2019/01/400_error_fiddler.png)


I started checking the test users and could confirm that the Token Seize had increased during migration to about 13k. I was looking for application logs on "App1" however, I could not find any Kerberos event log errors and there were not any errors on the domain controllers either. The domain controllers, Application servers and even the client had set max 64k as token seize.

![image](/wp-content/uploads/2019/01/kerberos-registry.png)

I started thinking that It could be related to the application having a limit on the http header, so I created a simple flow chart to confirm how the application worked before I asked the developer to change the HTTP header size in the Web config.

![image](/wp-content/uploads/2019/01/Kerberos400error.png)

### Summary

We started testing SSO after the web config HTTP was changed from 8k to 32k and Kerberos authentication was working like a charm again.  
  
Kerberos authentication can be quite interesting to troubleshoot, especially when the problem is related to multiple sources and you have to go down the chain to find the solution 🙂