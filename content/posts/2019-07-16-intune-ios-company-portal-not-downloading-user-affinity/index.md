---
weigth: 1
title: Intune – iOS Company Portal not downloading (user affinity)
author: Stein-Erik Alvestad
date: 2019-07-16T19:52:00+00:00
url: /intune-ios-company-portal-not-downloading-user-affinity/
categories:
  - intune

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: "seema-miah-32mF78M9GP4-unsplash.webp"

toc:
  auto: false

---

{{< admonition type=quote title="KEEP CALM AND COMPANY PORTAL WILL COME" open=false >}}
**quote**
{{< /admonition >}}




### Introduction

In 2019, I have been working on an MDM iOS migration project from Jamf to Intune. In this project, we got some problems regarding Intune and Company Portal (VPP) not being downloaded with **User affinity** Enrollment profiles for iOS. Trough this post I want to give some more insight/details regarding this issue, and how we &#8220;Solved&#8221; (workaround) it.

One of the technical requirements for Intune was to improve security and end-user experience, with automatically installing Company portal (CP) with Volume Purchase Program (VPP) during enrollment, and not requiring any Apple ID for all Business Application. To solve this we configure the correct Enrollment profile for iOS.

When configuring user enrollment profile for iOS with Intune we have some options to consider. With User affinity or without User affinity.  
Enroll with User Affinity with CP VPP will use the synced token from Apple DEP and for security, we use Company Portal as an authentication method



![Image](/wp-content/uploads/2019/07/Setup-Assistant-Customization_0.png)

At the first step, the user gets the Apple MDM profile given with the Setup Assistant. In my example, I just set the location services to minimize the clutter. It was important to not add any restore options because this could be a problem since Apple backup MDM profiles as part of the standard iOS backup. Enrollment is a mess with two MDM profiles if end-user enrolls the same iOS device.

![Image](/wp-content/uploads/2019/07/Setup-Assistant-Customization_1.png)
 

Since the devices are synced with Intune as supervised devices, they should get the CP automatically. So everything now looking good with our enrollment profile, and we are ready for enrollment at scale with all supervised iOS devices that are synced to Intune. We move the devices from the Apple DEP portal to Intune, and verify all devices are synced and have the correct status &#8220;last contacted &#8211; never&#8221;


![image](/wp-content/uploads/2019/07/enrollment_program_token_devices.png)

At this stage we are now ready for End-users starts to factory reset iOS device so that it can get a fresh MDM profile from Intune. The End-users start the enrollment starts and the user gets the MDM profile, however, Company portal is missing. Huh, strange!

### Troubleshooting

Time to start troubleshooting. The device is registered in Intune with status &#8220;Not Evaluated&#8221;, the device has also got the last sync status in the Devices status. We verify that we have enough CP VPP licenses. However, the license count in the available licenses does not decrease. We also sync the VPP token once more and verify that the Intune status page looks good.

![image](/wp-content/uploads/2019/07/Company-Portal-apps-VPP-1.png)

Checking the Monitor logs, we cannot see any good error codes that we could troubleshoot further. But the Enrollment Page failures are increasing with devices.

![image](/wp-content/uploads/2019/07/Device-enrollment_monitor.png)

As some users start to factory reset the devices for the second time, suddenly Company Portal is downloading to the device. Without seeing any errors in Intune we create a ticket with Microsoft providing them with details.

Meanwhile waiting for Microsoft to respond I&#8217;m looking further into the documentation from Microsoft. Looking further into CP VPP, we see that the app license is given to the users that enroll. Since it&#8217;s not any requirement from MS to assign the CP VPP to any Dynamic group it&#8217;s displayed as information &#8220;Assign application to at least one group. Click &#8216;Assignments&#8217; &#8220;, you automatically think something is wrong configured. But looking into the docs, I can see more details about how Intune creates a shadow dynamic group that we never seen in Azure Ad groups. This happens in Intune / Azure AD backend.

This is not easy to find in the Intune documentation, but can be found same can be found with Windows 10 and Autopilot documentation:

* docs.microsoft.com/en-us/intune/enrollment-autopilot#windows-autopilot-for-existing-devices

where they also mention DEP in the same documentation.  


{{< admonition type=info title="Microsoft documentation below:" open=true >}}
{{< /admonition >}}

---

Because the correlator ID is not pre-listed in Intune, the device may report any correlator ID they want. If the user creates a correlator ID matching an Autopilot or Apple DEP profile name, the device will be added to any dynamic Azure AD device group based off the enrollmentProfileName attribute. To avoid this conflict:  
&#8212; Always create dynamic group rules matching against the&nbsp;_entire_&nbsp;enrollmentProfileName value  
&#8212; Never name Autopilot or Apple DEP profiles beginning with &#8220;OfflineAutopilotprofile-&#8220;.

---

The part about DEP did not reflect our environment, but the process gives some information about the logical part of the assignment.

As we did not have much choice to wait for any solution we just had to continue our workaround enroll the iOS device for the second time so that the Device could get the CP. However 4 times during 1 month we had to sync the VPP token manually with intune because Intune never downloaded the CP VPP even after multiple factory resets.

After getting the Microsoft ticket to the product team, they were able to identify the behavior, and we got some more detail about the recommended solution. Most of our devices were already enrolled at that time, so the problem was not that big anymore, however during the period this was PITA from End-user perspective.  

### Summary
So back to the solution, Just wait 🙂 The patch from MS fixing CP will be in production hopefully at the end of July or start of August 2019. Will try to update the post when we confirm the problem is finally solved.

**Update** 
  
Intune tenant was updated in July and fixed our problem.  

**However**, we also got another strange CP problem in July,&nbsp; since Apple VPP token became invalid after the VPP account changed password.  
The logic if this cannot be explained with any of the documentation I have read.&nbsp; It&#8217;s ok that you will have to use a VPP account when adding the token, however, any password changes should not conflict with the token. I also created MS ticket for this problem, however, the root cause was never found.&nbsp;&nbsp;

&nbsp;

&nbsp;

### Useful links:

* https://docs.microsoft.com/en-us/intune/enrollment-autopilot#windows-autopilot-for-existing-devices