---
title: 0x80004005 office shared computer activation with Windows Virtual Desktop
author: Stein-Erik Alvestad
date: 2021-04-12T11:04:33+00:00
url: /0x80004005-office-shared-computer-activation-with-windows-virtual-desktop/
categories:
  - Office
  - WVD

---

### Introduction

I was recently involved in troubleshooting shared computer activation with Office on a WVD hostpool image using the latest 20h2-evd-o365pp-g2 image sku and Microsoft 365 Business Premium for Office.  
  
To start with the Troubleshooting this was the only error that occurred for end user trying to activate.  
  
account issue  
error code 0x80004005

![image](/wp-content/uploads/2021/04/code08004005_office_activation.png)


The Image was using &#8220;**20h2-evd-o365pp-g2&#8243;** from the Azure gallery image. So all the settings should be correct regarding Shared activation. And I could confirm this by looking into the Registry was applied with the correct **settings**:  

HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Office\ClickToRun\Configuration  
SharedComputerLicensing was set to 1. 

The Host pools Virtual Machines was also also behind strict firewall rule set, so I also verified that it could talk to the Microsoft Activation URL <a target="_blank" href="http://activation.sls.microsoft.com/" rel="noreferrer noopener">activation.sls.microsoft.com</a> on 443.  
  
My last assumption was check the licenses. I was informed that the users were assigned correct license with Microsoft 365 Business Premium that should support Multi-session WVD office version and Office Shared Computer Activation.  
  
However, I was just curious to check what features were enabled on the license SKU. And without a doubt the most obvious was not checked. The Microsoft 365 Business Premium feature to enabled &#8220;Office Shared Computer Activation&#8221; was set to off / disabled.

![image](/wp-content/uploads/2021/04/0x80004005_office_shared_license.png)

### Summary

Changing the License to &#8220;on&#8221; / &#8220;enabled&#8221; in the Group based license Enabled Services, solved the issue immediately!<figure class="wp-block-image size-large">

<img decoding="async" loading="lazy" width="253" height="70" src="/wp-content/uploads/2021/04/office_shared_computer_activation.png" alt="" class="wp-image-1457" /> </figure> 

The last step was to verify that the Licenses was &#8220;activated&#8221; in the user profile Appdata folder in the following location:  


%localappdata%\Microsoft\Office\16.0\Licensing

![image](/wp-content/uploads/2021/04/License_solved-1024x176.png)

