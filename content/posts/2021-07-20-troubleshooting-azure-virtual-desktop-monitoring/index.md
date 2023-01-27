---
weigth: 1
title: Troubleshooting Azure Virtual Desktop Monitoring
author: Stein-Erik Alvestad
date: 2021-07-20T09:45:41+00:00
url: /troubleshooting-azure-virtual-desktop-monitoring/
categories:
  - Azure Monitor
  - Azure Virtual Desktop

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: "tobias-tullius-4dKy7d3lkKM-unsplash.jpg"

toc:
  auto: false
---


### Introduction

I was recently involved in a project to troubleshoot some performance issues on Azure Virtual Desktop related to some users who reported connection error **&#8220;The remote resource can&#8217;t be reached&#8221;** Check your connection and try again&#8221; when connecting to Azure Virtual Desktop. When the user connected 2-3 times they could eventually connect. 

So I naturally started the troubleshooting by looking into Azure Monitor Insights for Azure Virtual Desktop.  
But when opening the Azure Monitor Dashboard I did not get access to the Utilization, Host Diagnostics, Host Performance, or Alerts. I could however query other tabs like connection diagnostics, connection performance, Users, Clients. 

When trying to query data from Host Diagnostics the tab would start looping with the spinning wheel before it timed out, and the workspace reported: **&#8220;could not query this data&#8221;.** 

![image](/wp-content/uploads/2021/07/azure-monitor-take-it-for-a-spin-1024x324.png
)

I first suspected we could have some wrong configured counters, so I started looking the azure documentation

* https://docs.microsoft.com/en-us/azure/virtual-desktop/troubleshoot-azure-monitor#my-data-isnt-displaying-properly>

I looked into the performance counters and event logs to verify that they were correct. The Log Analytics workspace also did report **&#8220;no missing performance counters&#8221;**. I also double-checked the Terraform code, and could verify all counters were correct. Apparently, the problem was related to something else. 

the next step was to jump over to the session host and check the event logs. In the event log, I could see NOT Accessible URL&#8217;s with the Event ID 3702

![image](/wp-content/uploads/2021/07/image.png)

I did know that the session hosts was in a restricted network zone , so I got the network team to verify the firewall ruleset and we were missing only one URL suffix related to log analytics, but that was not relevant to the URLs from the Event ID above. Everything looked good in the firewall logs. 

Since we still had the same issues I went into the log analytics workspace to try to Query a simple test to see if the computers were listed. I did not get any results back.

![image](/wp-content/uploads/2021/07/image-1-1024x332.png)


Next step was to verify that the Monitoring Agent had connection the C:\Program Files\Microsoft Monitoring Agent\Agent\TestCloudConnection.exe.

![image](/wp-content/uploads/2021/07/TestCloudConnection-1-1024x280.png)

In addition, I viewed the Monitoring agent that reported status:  
**DNS name resolution of the Microsoft Operations Management Service Suite Failed. This Could be due to either the Workspace Id being configured incorrectly, or the agent not have internet access, please check that the system either has internet access&#8230;.**

![image](/wp-content/uploads/2021/07/Agent_error-1024x166.png)

## Solution

At this time It was definitely time to check **DNS**! 

I started checking ping request against some of the URL&#8217;s and could see that ping rdbroker.wvd.microsoft.com  
Ping request could not find host rdbroker.wvd.microsoft.com. Please check the name and try again.

After checking the DNS servers, I could see that the forwarders were pointing to some old DNS server that was due to be decommissioned and the firewall behind the old servers was terminating the traffic. We updated the forwarder to the Azure DNS server and our traffic started flowing correctly.

![image](/wp-content/uploads/2021/07/DNS_forwarder_to_azuredns-1.png)

I also did a quick test with the TestCloudConnection.exe, and the connectivity test passed as expected. 

![image](/wp-content/uploads/2021/07/image-2.png)

### Summary

This was not your everyday problem, but I just wanted to share some of the steps related to troubleshooting.

We have the Operational issue that reports status OK, but this option could be confusing because Log Analytics will report the agents as OK because is only checking the heartbeat connection between the workspace ID and the Key&#8217;s. 

What could have been great is that Azure Monitor could have functionality that reported that the agent was not OK directly from the Overview tab. 

Until next time! It&#8217;s always DNS

![image](/wp-content/uploads/2021/07/always_DNS.jpg)