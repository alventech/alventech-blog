---
weigth: 1
title: Yubico with Azure AD MFA
author: Stein-Erik Alvestad
date: 2019-03-14T10:10:09+00:00
url: /yubico-with-azure-ad-mfa/
categories:
  - Azure

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: ""

toc:
  auto: false

---



Yubikey 5C and Yubikey NFC

I recently bought the Yubikey 5C and Yubikey NFC from yubico.com.  
Yubico is in short summary a company behind Yubikey hardware auth device supporting (OTP / FIDO2) protocols. You can read much more information and details at yubico.com. 

Since I&#8217;m Interested</g> In Security and Identity authentication, I wanted to do more testing with **Azure MFA for OATH hardware tokens** (public preview) and **Windows 10 Passwordless** (private preview). I&#8217;m also looking into the use of NFC, so that&#8217;s why Yubikey NFC for iOS device. LastPass is the only App I have found that leverages this functionality. Hopefully, there will be more apps will get this as developers make use of the SDK. In this guide I will go trough the steps of **Azure MFA for OATH hardware tokens** (public preview)  


Quick FAQs Yubikeys with Azure MFA

Configure Azure MFA for OATH hardware tokens (public preview) 

### Quick_FAQs_Yubikeys_with_Azure_MFA

  * Only public preview for AAD
  * Works with Azure cloud MFA even though it&#8217;s in Azure MFA settings of the AAD portal. 
  * AAD Supports OATH-TOTP SHA-1 Tokens (30 or 60 sec)
  * AAD Only supports 3 Yubikeys, one MS Authenticator app, phone for each user account.

### Configure Azure MFA for OATH hardware tokens (public preview)  
  
#### Prerequisites

Azure tenant with AAD Premium
  * MFA already enabled
  * Have at Least 1 Yubikey. For the purpose of this guide, I&#8217;m using a [Yu][1][[bikey 5C][1]][1]  [][1]
  * Yubico Manager Command (to extract Yubikey Serial number)
  * Yubico Authenticator 

###  Configure Yubikey

  
**Download and install** ( Windows 10, MacOs or Linux)  

  * [**Yubico Manager Command**][2] 
  * [**Yubico Authenticator**][3] 

Start the Yubico Manager Command from terminal/powershell.  

**To get the Serial Number of YubiKeys.** Use command 

.\**ykman**.exe List 

**To link TOTP key for Yubikey**. Use command  


.\**ykman**.exe **oath** **add** UPN@<**tenant** name>onmicrosoft.com 

Next step requires you to add a Base32 Key.  
Use OpenSSL to create a Base32 key.  
Use to generate Base32 key if you have OpenSSL on your pc.  

Copy your Base32 Key to after you have run the \**ykman**.exe command


![Image](/wp-content/uploads/2019/03/y_Windows-PowerShell.png)

### Step 2) Configure Azure MFA

Go to the AAD portal and go to MFA server. In settings go to OATH Tokens. Choose Upload CSV to Azure.  
  
The CSV has to be in a specific format like the example below  
  
upn,serial number,secret key,timeinterval,manufacturer,model  
UPN@.onmicrosoft.com,1234567, 1234567890abcdef1234567890abcdef,30,YubiKey,HardwareKey  

![Image](./OATH-tokens-Upload.png)


After the Upload has successfully completed, go to the refresh button.  
Choose Activate. 

![Image](./OATH-tokens-Activate-AzureAD.png)

Verify that the YubiKey is Activiated in the dashboard.

![Image](./
OATH-tokens-Activatevated-AzureAD-1024x274.png)



### Step_3_Configure_MFA_settings_on_the_User

Go to https://aka.ms/mfasetup with the user who has been assigned the YubiKey. In the settings, change the preferred auth Options to Use verification code from 

You can see that the user has already been assigned the Yubikey token. Enter the token from the Yubico 

Autenticator on your pc.  


![Image](./
add_sec-550x578.png)


Logout from MFA portal. Start a new login to portal.office.com  
you will now see that you will get prompted with autenticator
(Yubikey must be plugged in to get OTP) and verify that is works.

![Image](./
YubiSigning.png
)


### Summary

As stated this is only a public preview, and currently only works with  
OATH-TOTP SHA-1 and you need the Yubico authenticator app. So it would have been nice to see if these can go next step using the HTOP and Password-less. In comparison with other Hardware tokens, Yubico has some competition from token2.com and deepnetsecurity.com, and I will look into these later:)  
  
As part of the deployment process, I could have used Self Service Hardware token setup, but this limit the administrator part to track all the Hardware keys.  
  
It&#8217;s still a lot of potentials in this space, so it will be exciting to see how this develops 🙂

Useful links:

* https://www.yubico.com/product/yubikey-5c/#yubikey-5c
*  https://developers.yubico.com/yubikey-manager-qt/
*  https://www.yubico.com/products/services-software/download/yubico-authenticator/