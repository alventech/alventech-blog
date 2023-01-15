---
title: Azure Virtual Desktop Published Application and Scaling issues with DPI
author: Stein-Erik Alvestad
type: post
date: 2021-07-21T10:44:20+00:00
url: /azure-virtual-desktop-published-application-and-scaling-issues-with-dpi/
categories:
  - Azure Virtual Desktop

---

![image](/wp-content/uploads/2021/07/michael-maasen-AkYGy_ymFqo-unsplash-1024x660.jpg)



### Introduction

    To scale or not to scale, thats todays question with Azure Virtual Desktop


Lately, I have had a compelling problem with Azure Virtual Desktop where a customer had problems with scaling a Line of business application (LOB) as a published application. Text, icons, and fonts became blurry, especially for users who only used laptops as the primary screen, and changing back and forth between primary and secondary screens with different resolutions & screen scaling. 

When first came into the project, the workaround with the GPO & registry for **IgnoreClientDesktopScaleFactor** and **LogPixel** from the url below was already tested but did not have any effect.

* https://cloud.accigo.se/blog/windows-virtual-desktop-scaling-issues/

After this step, I started testing a GPO with GDI DPI as an option. GPI DPI information from Microsoft <https://blogs.windows.com/windowsdeveloper/2017/05/19/improving-high-dpi-experience-gdi-based-desktop-apps/>

And Since GDI DPI GPO did not apply for the LOB. The next step was to look into the DPI Awareness of the application, so I l used Process explorer on the session host and added the DPI Awareness tab nr 1.

![image](/wp-content/uploads/2021/07/image-5-1024x297.png)

Comparing this with nr 3 explorer.exe and nr 4 msedge.exe (Per-Monitor Aware) that was also published as applications as well, I could see that our option nr 2 LOB.exe was not having any option for DPI Awareness, and that&#8217;s why GDI DPI did not apply either. 

### Overview_Client_Scaling_and_RDP_Properties

So without doing any big changes to the code in the application, I started playing with RDP Properties in AVD to see if we could make the scaling better. We did use both Windows desktop and Microsoft Store client apps. 

Summary Client features (win10) and Display settings in the table below:  

* https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-features>

* https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/rdp-files#display-settings><figure class="wp-block-table is-style-regular">

| Feature                      | Windows  
Desktop | Microsoft Store | RDP Display setting | Description                                                                                                                                                                                                         |
| ---------------------------- | ----------------- | --------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Remote Desktop sessions      | x                 | x               | x                   | Desktop of a remote computer presented in a full screen or windowed mode.                                                                                                                                           |
| Immersive RemoteApp sessions |                   | x               | x                   | Individual remote apps presented in a window or maximized to a full screen.                                                                                                                                         |
| Dynamic resolution           | x                 | x               | x                   | Resolution and orientation of local monitors is dynamically reflected in the remote session. If the client is running in windowed mode, the remote desktop is resized dynamically to the size of the client window. |
| Smart sizing                 | x                 | x               | x                   | Remote Desktop in Windowed mode is dynamically scaled to the window&#8217;s size.                                                                                                                                   |</figure> 

To verify the client setting on the end-user client I used both Windows Desktop Client and the Microsoft Store Client.  
NB! To check that RDP custom settings have been applied, I checked the logs on the local client.

**C:\Users\%username%\AppData\Local\rdclientwpf** and the folder contain the RDP settings.

I tested with Dynamic resolution and smart sizing. But non of the settings worked for the LOB Application. Using the Microsoft Store Application actually just made scaling even worse in the combination with RDP settings.  


### Solution

Unfortunately, The RDP settings did not help, so the last step was to work with the application vendor to see if we could improve the DPI. After some days of tinkering with different settings like updating the .NET version of the application and changing back and forth with pixels, we could publish a new Remote Application with the new settings. And with new a week of testing, we finally ended up with some settings that were acceptable for the end-users! 

### Summary

When doing application assessment for migration from Citrix to Windows 10 multi-session host in Azure Virtual Desktop you should consider DPI as part of your scope. DPI could have a huge impact on the end-users and their workflows. And if you are not as lucky as I was with the application vendor. This could end up being a very time-consuming problem.