---
weigth: 1
title: FEITIAN FIDO Keys for Passwordless
author: Stein-Erik Alvestad
date: 2019-08-28T20:57:50+00:00
url: /feitian-fido-keys-for-passwordless/
categories:
  - AAD
  - Feitian
  - Passwordless
  
lightgallery: true

images: []
resources:
- name: "featured-image"
  src: "202211211511076827.jpg"

toc:
  auto: false

---
### Introduction

I have gotten some new FIDO keys from FEITIAN, and have done some testing to see how they work with Azure AD and Passwordless.  
So this Blogpost is more like the following up series, from the last post 

### Heard about Feitan ?

I certainly never heard about them before, and first came across FEITIAN, when Microsoft announced the partners for FIDO support.  

* https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Microsoft-passwordless-partnership-leads-to-innovation-and-great/ba-p/566493

Looking more into FEITIAN FIDO support, it sure looked interesting since they have the possibility for Biometrics like Fingerprint. I ordered the FIDO2 Bundle (K33 + K27 + K26) from 
* http://www.ftsafe.com/AzurePublicPreview

Shipping from China to Norway with FedEx was about 1-week.

The full FIDO specification is available:  

* https://www.ftsafe.com/Products/FIDO/Bio

Unboxing the FEITIAN FIDO2 Bundle (K33 + K27 + K26), First impressions are premium, the build quality feels good.

![image](/wp-content/uploads/2019/08/20190815_143348445_iOS-2.jpg)



### Configure K33 with Windows 10

The K33 from the right above supports Bluetooth, however since Bluetooth is basically broken when comes to security, I would absolutely think twice before using this at scale!

I used guide below for configuring the K33.
* https://www.ftsafe.com/Support/Resources 

The first-time setup requires to hold K33 Bluetooth pairing key for about 5 seconds. The device also supports USB-C (cable not included)  
After pairing, go to Windows 10 &#8211; Sign-in Options.&nbsp; Security Key. Manage and ADD your Fingerprint.  
Now go to https://myprofile.microsoft.com and add your FIDO Security Key.

I captured a video to showcase the FEITIAN K33&nbsp; work with Windows 10 version 1903

{{< youtube fNkRUw8YYwE >}}

### Azure AD Key Restriction Policy

Given the fact that there are other FIDO providers, it&#8217;s important to consider your trust, and think about what keys your organization will support.  
Azure AD will support this feature using the KEY Restriction Policy. This is done with Authenticator Attestation GUID (AAGUID).

Read more about AAGUID, that&#8217;s well-done documented  
* https://fidoalliance.org/specs/fido-v2.0-rd-20180702/fido-metadata-statement-v2.0-rd-20180702.html

![image](/wp-content/uploads/2019/08/KeyRestrictionFIDO-1.png)

Use Coupon CODE Stein-20 when you buy, this will support my channel and give you some discount! 🙂 You can buy FEITIAN products at 

https://www.ftsafe.com/store/

That&#8217;s all for this short blog post  
  
Have a passwordless day 🙂


* https://fidoalliance.org/specs/fido-v2.0-rd-20180702/fido-metadata-statement-v2.0-rd-20180702.html