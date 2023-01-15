---
title: Passwordless with Windows 10 and Yubikey
author: Stein-Erik Alvestad
date: 2019-07-25T19:00:55+00:00
url: /passwordless-with-windows-10-and-yubikey/
categories:
  - AAD
  - intune
  - yubikey
tags:
  - Passwordless with Windows 10 and Yubikey; Passwordless Windows 10

---
![image](./tmnt-secure.png)

Look at these guys! They are so happy, they have gotten their Yubikey’s and are ready for some Passwordless Authentication with Windows 10.


### Introduction

In March I posted a blog post about using Yubikey with Azure AD, 

So this post is a follow-up since Microsoft now has support for passwordless and Fido2 with Windows 10 **(still only preview)**.  

In his post, I will go through the steps regarding configuring passwordless in Intune, Azure AD and Windows 10 with Yubico.  
For the purpose of this guide, I will use Windows 10  version 1903 since it has more capabilities in the Policies like users can change their PIN, update biometrics, or reset their security key, and does not require tool from Yubico.

  Quick FAQs Passwordless with Windows 10
  
  Step 1) Enable Windows Hello for Business
  Step 2) Intune OMA-URI for Security Key
  Step 3) Enable combined registration experience

Step_4_Enable_new_passwordless_authentication_methods

Step 5) Add the FIDO key to the user profile
Step 6) Testing Passwordless with yubikey’s on Windows 


**Quick FAQs Passwordless with Windows 10**

  * Still Only Public Preview
  * Working from Windows 10 1809, however, 1903 is recommended for testing.
  * Azure AD (MFA) must be configured
  * Combined registration required
  * FIDO2 key like Yubikeys or other from MS documentation.
  * Supports WebAuthN in Edge Only
  * The FIDO2 “Key restriction policies” do not work yet.
  * FIDO2 support for AAD Hybrid Joined is not supported yet. MS working on support for Hybrid ( Hopefully supported beginning of Q1 2020)

### Step 1) Enable Windows Hello for Business

Go to[&nbsp;https://devicemanagement.microsoft.com][1]  
Choose the Windows Hello for Business settings in Intune. Change the Use Security keys for sign-in to &#8220;**Enabled**&#8220;

**&nbsp;**_If you haven&#8217;t configured Windows Hello for Business. You will need to enable this setting as well._  
_Windows Hello is a topic by itself, so I recommend reading the documentation from Microsoft,&nbsp; if you need to look into the different settings and scenarios like Hybrid, Cloud only or on-premise <https://docs.microsoft.com/nb-no/windows/security/identity-protection/hello-for-business/hello-planning-guide>_<figure class="wp-block-image">

![image](./Device_Enrollment_enable_security_Keys_sign-in-1024x538.png)


### Step 2) Intune OMA-URI for Security Key

Create the OMA-URI settings in Intune &#8211; Device Configuration &#8211; Profiles &#8211; Create Profile and Add Custom OMA-URI for Windows 10.

**Name**  
&#8220;Name it whatever you want&#8221;

**Description**  
&#8220;Name it whatever you like&#8221;

**OMA-URI**  
&#8220;./Device/Vendor/MSFT/PassportForWork/SecurityKey/UseSecurityKeyForSignin&#8221;

**Data type  
** &nbsp;&#8220;Integer&#8221;

**Value**  
&#8220;1&#8221;

_If you think is strange this OMA-URI setting is called Passport for Work, it&#8217;s just the old name before it was named Windows Hello for Business.&nbsp;_<figure class="wp-block-image">

![image](./OMA-URI-Hello-Passwordless.png)

I will now add the Profile to my Pilot target group.<figure class="wp-block-image">

![image](./grp-passwordless-assigment.png)



### Step 3) Enable combined registration experience

Now let&#8217;s go back to Azure AD &#8211; <https://aad.portal.azure.com>

Azure AD Manage section &#8211; User settings and choose the Manage user feature preview settings

Change it to, **Users can use preview settings** to select the group for your Pilot users

![image](./user_settings_azure_ad.png)




### Step 4) Enable new passwordless authentication methods

Back to Azure AD Security section &#8211; <https://aad.portal.azure.com AAD> – Auth methods. Authentication method Policy (preview) blade.

Choose the &#8221; Click here to **Enable** users for the enhanced registration preview&#8221;. If its not already enabled.  
Add your user target group and save.

![image](./auth_methods-1.png)

Next, go to Methods and **Enable** FIDO2 Security Key.

_The Enforce key restriction is not supported by Microsoft yet . Will be before GA.&nbsp;_<figure class="wp-block-image">

![image](./FIDO2-security-settings-auth-metodhs.png)

### Step 5) Add the FIDO key to the user profile

Go to the <https://myprofile.microsoft.com>

Add FIDO key to user account with the user account in the Security Info tab on the left side.

![image](./seuciryt-info.png)

If you this message. You will have to wait or verify your user&#8217;s group permissions.  
It took some time for me before the feature was enabled for the user in Azure AD.

![image](./oops.png
)


### Step 6) Testing Passwordless with yubikey’s&nbsp; on Windows 10**

We are now ready to test on a Windows 10 Version 1903 Computer.

I can see that the OMA-URI has pushed the policy with login option.  
Just need to add my Yubikey, Add my Yubikey PIN and Tap the Yubikey to login like the screenshot below.

![image](./tap.png
)


### Summary

Version 1903 gives a better user experience since 1809, however, from an end-user perspective I don&#8217;t think the dongle is for everyone. But this improves security and there are multiple use cases for this in the filed, so it&#8217;s not strange that Microsoft is pushing to improves Passwordless Experience throughout 2019. When the big players like Apple and Google adding Fido into the Android and iOS devices, the journey to test Passwordless this is even more exciting. WebAuthn also has a lot of potentials and this will expand with browsers capabilities.

In my lab, the next step is still to have a device with NFC so that I could test this functionality.

#### Further reading

*  https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless

* https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Announcing-the-public-preview-of-Azure-AD-support-for-FIDO2/ba-p/746362

* https://docs.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-passwordless-enable

* https://techcommunity.microsoft.com/t5/Windows-IT-Pro-Blog/Expanding-Azure-Active-Directory-support-for-FIDO2-preview-to/ba-p/981894

To get the latest news and updates on Passwordless with Windows 10 and Azure AD follow @swaroop_kmurthy
@AlexSimonson Twitter

* https://devicemanagement.microsoft.com
* https:/twitter.com/swaroop_kmurthy
* https://twitter.com/Alex_A_Simons