---
weigth: 1
title: Bitwarden and Yubikey
author: Stein-Erik Alvestad
date: 2020-07-22T09:16:42+00:00
url: /bitwarden-and-yubikey/
categories:
  - Bitwarden Password Manager
  - yubikey
tags:
  - Bitwarden
  - NFC Yubikey
  - Yubikey

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: ""

toc:
  auto: false

---

### Introduction

Password Managers is still a necessity in 2020, and will be for a long time. I&#8217;ve been using Bitwarden with YubiKeys for a while now, so it was about time to share some experience, and how easy it&#8217;s to get started. 

Bitwarden <a href="https://bitwarden.com/" target="_blank" rel="noreferrer noopener">https://bitwarden.com/</a> is an Open Source Password Manager that is hosted on GitHub <a aria-label="undefined (opens in a new tab)" href="https://github.com/bitwarden" target="_blank" rel="noreferrer noopener">https://github.com/bitwarden</a>. This means that you have full control over source code, and you also could contribute to the project. With Bitwarden you can simply self-host it, or run their Organization Plans that is hosted on Azure <a aria-label="undefined (opens in a new tab)" href="https://bitwarden.com/help/article/cloud-server-security/" target="_blank" rel="noreferrer noopener">https://bitwarden.com/help/article/cloud-server-security/</a>. Bitwarden offers different Organization Plans. To see what fits your demand go to <a aria-label="undefined (opens in a new tab)" href="https://bitwarden.com/#organizations" target="_blank" rel="noreferrer noopener">https://bitwarden.com/#organizations</a>. The Enteprises Plan supports Azure AD integration. I will cover this in another blog post.  


### Configure Bitwarden with your Favorite FIDO key

Bitwarden supports a whole range of Providers. When enabling 2FA you will also be prompted to download recover Code. Download these first, before proceeding to the 2FA setup! And **Keep the Recovery codes safe**, in case things went south! 

To configure your FIDO key with YubiKey.  
  
Go to:  
1) Settings  
2) Two-step Login  
3) Manage

![image](/wp-content/uploads/2020/07/image-7-1024x741.png)


Just add your YubiKey with the Touch/Tap were it will populate the OTP or U2F, I recommend at least two YubiKeys, in case you lose your primary key. With the second backup key you can access your account if you ever lost your key! Yes, that will eventually happen 🙂 You could also configure Bitwarden with another supported OTP/U2F key if you have another brand in your possession. If you do not have one, Authenticator Apps like Authy, Google, Duo is supported as well. 

The list of supported Yubico keys that will works with Bitwarden 
* https://www.yubico.com/works-with-yubikey/catalog/bitwarden-premium/

If you need to support an NFC compatible key, use the NFC support. I have NFC supported key, so I can access the Bitwarden on go with my Phone.

![image](/wp-content/uploads/2020/07/image-2.png)


### Test Login

After you have saved the settings, it&#8217;s time to test the login from your browser of preference. The login site&nbsp;[https://vault.bitwarden.com][1]

![image](/wp-content/uploads/2020/07/image-6.png)

![image](/wp-content/uploads/2020/07/image-5.png)


Let&#8217;s also test a login with iOS with an NFC compatible YubiKey

### Summary

It&#8217;s without doubt that configuring 2FA with Bitwarden is straightforward with some simple steps. Just remember! Do not only configure one single YubiKey, that is not good practice.

Bitwarden is feature-packed and should tick all your boxed if you have strong requirements for a password manager. With the wide range of supported OTP/U2F, I think this password manger will make you delighted for a long time. And yes, it&#8217;s Open Source that is a big bonus! 

### Documentation

  * <a aria-label="undefined (opens in a new tab)" href="https://bitwarden.com/help/article/setup-two-step-login-u2f/" target="_blank" rel="noreferrer noopener">https://bitwarden.com/help/article/setup-two-step-login-u2f/</a>
  * <https://www.yubico.com/works-with-yubikey/catalog/bitwarden-business/>

* https://vault.bitwarden.com/