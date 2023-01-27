---
weigth: 1
title: Intune – MEM – Configure Edge browser for iOS and Android
author: Stein-Erik Alvestad
date: 2020-02-23T17:30:00+00:00
url: /intune-mem-configure-edge-browser-for-ios-and-android/
categories:
  - Android
  - EDGE
  - intune
  - iOS
  - MEM

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: "retried_browser.png"

toc:
  auto: false
---

### Introduction

It's Time to Move to EDGE Mobile! 
 
**Step 1** App Protection Policies
  Target Edge for iOS and Android

**Step 2** App configuration polices &#8211; Target Edge for iOS and Android</a>

**Step 3** Check out new Browser experience with EDGE</a>



Documentation

### Its_Time_to_Move_to_EDGE_Mobile

Back in November 2019 the Managed browser was announced to be retired March 31 2020. Already from February 1, 2020, the Intune Managed Application was going to removed from Google Play Store and iOS App Store. Since March is right around the corner, it&#8217;s about time to get prepared to do the switch. 

Regarding a smooth transition, Microsoft let us use all the same Browser Config settings like &#8220;com.microsoft.intune.mam.managedbrowser&#8221;, so basically we just need to target the Edge for iOS and Android with the existing MAM Policies (app protection and app configuration settings). 

With this blog post, I will cover the steps to get started and deep into the browser experience. And In my particular case, I&#8217;m going to use EDGE iOS as a corporate browser with some specific settings, just to show the user browser experience. We are also checking the custom settings like &#8220;Single sign-on&#8221;, Bookmarks and Blocking some specific sites from both end-user and MEM side.  
  
Settings:

  * [Specify allowed or blocked sites list for Microsoft Edge][1]
  * [Configure managed bookmarks for Microsoft Edge][2]
  * [Transition users to their personal context when trying to access a blocked site][3]

#### Step 1 ) App Protection Policies &#8211; Target Edge for iOS and Android

To get started we need to look into the existing App protection Policy. 

![image](/wp-content/uploads/2020/02/2020-02-16-13_52_31-Apps-App-protection-policies-Microsoft-Endpoint-Manager-admin-center.png)



Add EDGE to your Policy under Apps &#8211;> Edit and choose EDGE as a Public App (I&#8217;m Adding both iOS and Android).

![image](/wp-content/uploads/2020/02/02-Intune-App-Protection-Properties-Microsoft-Endpoint-Manager-admin-center-1024x598.png)




#### Step 2) App configuration polices &#8211; Target Edge for iOS and Android

Change the App configuration polices for your particular configuration.

![image](/wp-content/uploads/2020/02/image.png)

Add EDGE to your Policy under Basics &#8211;> Edit and choose EDGE as a Public App (I&#8217;m Adding both iOS and Android). 

![image](/wp-content/uploads/2020/02/1Targeted-App-Config-Properties-Microsoft-Endpoint-Manager-admin-center.png)



#### Step 3 ) Check out new Browser experience with EDGE

After adding the EDGE Application to the Policy, we can see that we now have a new layout. All settings have been merged to the &#8220;**EDGE Configuration Settings**&#8221; where the &#8220;com.microsoft.intune.mam.managedbrowser&#8221; is in the new view. You can see that is much easier to change settings without knowing all the &#8220;com.microsoft.intune.mam.managedbrowser&#8221; settings. We also have new settings like Import and Export either Blocked URLs which is a welcome addition. <figure class="wp-block-image size-large">

![image](/wp-content/uploads/2020/02/Edit-app-configuration-policy-MEM-1024x764.png
)

In the video below you can see how the MAM policy is applied and how the browser experience looks when MAM configurations are applied.

#### How does &#8220;_Single sign-on_&#8221; work on EDGE with iOS?  


I have not found any documentation regarding how the &#8220;Single Sign-on&#8221; Flow works. So to find how &#8220;SSO&#8221; was working with iOS, I had to dig into the EDGE browser [logs][4]. In the logs, I could find that EDGE is using Cross-app/&nbsp;keychain sharing with Company Portal, and the SSO&#8221; flow is using ADAL &#8211; Azure AD v1.0 endpoint. So regarding the authorization flow, the Company Portal already has a valid Authorization Token, and the next request against Azure AD v1.0 endpoint will use the same Application ID. The EDGE Application has another redirect URI than the Company Portal application, but keychain sharing is part of the SDK, so it will use ADAL token. The figure below illustrates the Authentication Flow from 1 to 4. I&#8217;m guessing this will change over time since Microsoft will probably migrate to using MSAL &#8211; Azure AD v2.0 endpoint eventually. 

![image](/wp-content/uploads/2020/02/Auth_flow_cross_app_ios_edge_CP-1024x295.png
)



#### Summary

The technical changes regarding adding EDGE as the new shiny browser is lucky for us not much work. The needed work will be to create new guides and information to the End-Users. During the pilot, I found one handy key feature when running both browsers for a period of time is that we can [Direct users to Microsoft Edge instead of the Intune Managed Browser][5] with custom settings.  
  
In terms of new MAM Configuration settings, I hope that we will get some more control like changing the start page, so it automatically starts the browser like it did in the Managed Browser. I could also like that we have more control over the SSO settings so it Automatically starts the Sign-On view right after MAM Policy is applied. 

### Documentation

  * <https://docs.microsoft.com/en-us/intune/apps/manage-microsoft-edge>
  * <https://techcommunity.microsoft.com/t5/intune-customer-success/use-microsoft-edge-for-your-protected-intune-browser-experience/ba-p/1004269>
  * <https://docs.microsoft.com/en-us/azure/active-directory/azuread-dev/howto-v1-enable-sso-ios>

* https://docs.microsoft.com/en-us/intune/apps/manage-microsoft-edge#specify-allowed-or-blocked-sites-list-for-microsoft-edge
*  https://docs.microsoft.com/en-us/intune/apps/manage-microsoft-edge#configure-a-homepage-shortcut-for-microsoft-edge
*  https://docs.microsoft.com/en-us/intune/apps/manage-microsoft-edge#transition-users-to-their-personal-context-when-trying-to-access-a-blocked-site
* https://docs.microsoft.com/en-us/intune/apps/manage-microsoft-edge#use-microsoft-edge-on-ios-to-access-managed-app-logs
* https://docs.microsoft.com/en-us/intune/apps/manage-microsoft-edge#direct-users-to-microsoft-edge-instead-of-the-intune-managed-browser