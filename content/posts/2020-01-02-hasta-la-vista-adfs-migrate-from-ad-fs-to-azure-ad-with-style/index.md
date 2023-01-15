---
weigth: 1
title: Hasta La Vista ADFS – Migrate from AD FS to Azure AD with style
author: Stein-Erik Alvestad
date: 2020-01-02T10:00:21+00:00
url: /hasta-la-vista-adfs-migrate-from-ad-fs-to-azure-ad-with-style/
categories:
  - AAD
  - adfs

---

### Introduction

Microsoft introduced AD FS application activity report (preview) and Azure AD staged rollout (preview) back in November 2019. These announcements are great opportunities to start the planning year 2020, to get rid of your AD FS environments. I will throughout this post see if these tools can speed up the migration processes.  
  
Why start this project when my ADFS environment is working &#8220;perfectly&#8221; and our organization has spent so much time configuring setting up the 3.party integrations. Well, there are some really big benefits of moving to Azure AD. Let&#8217;s start with some low hanging fruits related to the AD FS infrastructure that we can get rid of; 

  * Patching AD FS farms 
  * Capacity planning 
  * High-cost on-prem services like ADFS/WAP Farms, SQL AG, Load balancers
  * 3.party MFA extensions
  * Maintenance of certs Public SSL and AD FS Token Signing
  * Skilled resources to troubleshoot complex issues. 
  * Monitoring all Infrastructure components above. 

When it comes to cost, the cost is being moved over to the license plans in the AzureAD subscription Model, requiring either P1/P2. But with better security functionality I think it is absolutely worth looking into the switch. 

As most organizations have quickly moved to Office 365 and Azure AD the last years, still many customers use on-prem federations services like AD FS, however, now that Azure AD has matured a lot the last 4 years, we should use the benefits of Azure AD-like;

  * Password hash synchronization
  * Conditional Access 
  * Block legacy Auth
  * Identity Protection
  * Passwordless
  * Leaked Credential detection
  * B2B
  * Entitle Managment
  * Azure AD MFA
  * Better Control over Governance 
  * plus so much more good stuff! 

### Stage 1 &#8211; Plan AD FS Application activity report  {.wp-block-heading}

So to plan an assessment of our exiting environment, we can start to get data into the AD FS Application activity report. 

Azure AD Usage and insights reporting is the console showing us ADFS Application Activity. To get Data from the AD FS environment we need to install health Agents. With the agent, we are able to discover the ADFS Applications that can be migrated. 


![image](./ADFS-activity-page-1024x613.png)

Download the AdHealthAdfsAgentSetup.exe and install the agent on your AD FS servers. Login with Azure AD credentials to configure the agent. 

![image](./ADFSHealthAgentInstall.png)


To verify that our setup is ok, I like to verify that our network traffic is running smoothly. So to test this we run Health Agent Powershell module with the command below: 

```
Test-AzureADConnectHealthConnectivity -Role 
```

![image](./ADFSHealthAgentInstall3-1024x400.png)

I also recommend verifying that the agents and the ADFS auditing are enabled correctly. Go the Status Page In Azure Console: 
* https://portal.azure.com/#blade/Microsoft\_Azure\_ADHybridHealth/AadHealthMenuBlade/AdfsServicesList  


![image](./adfsADhelathpage-1024x295.png)


Since I was missing AD FS Auditing I got a warning from the Status Page. It&#8217;s a prerequisite to enable Auditing on the ADFS server.  
The Agent will report back when this is enabled. Be patience at this step, because it can take some time before the status changes to Solved. 

![image](./ADFS-auduting-resolved.png)

When the requirements above are in place, we will get data into ADFS Application Activity &#8211; Usage and insights
*  https://portal.azure.com/#blade/Microsoft/AAD/IAM/EnterpriseApplicationsInsightsMenuBlade/AppMigration

![image](./400_error_fiddler.png)

Usage-insights-Preview_BLADE-1-1024x322.png

![image](./ADFS_RP_TRUST-1024x211.png)


We can see that the list is just a summary of the applications that can be found in the AD FS Relying Party Trust list. But with Usage and insight, we now get additional details related to the Application Identifier, Unique User Count and Migration Status. The Migration status part will look into the Claims and if the Application is Ready for Azure AD. 

Let us start with a Migration ready Application like &#8220;Workplace by Facebook&#8221;. Here we see that the status has an application ready, however, the &#8220;**Create a new application**&#8221; is not ready in the preview yet. There is an open issue regarding this on the GitHub documentation Library:

* https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/migrate-adfs-application-activity

In this particular application, I know that it&#8217;s better to use the Workplace by Facebook Application from Azure AD Gallery, so it will be interesting to see how this functionality will work eventually. 

![image](./MigrationReadyMigrationdetails-1024x901.png)

Now we can look into the application Amazon Web Services that has the Warning with Additional Steps Required. This is where I think the most value of the AD FS Application activity report comes. 


![image](./amazon_webservices-Migration-detail-1024x926.png)



Looking more into the Potential Details we could see that we have some issues with 3 claim rules, and if I go into the Rule Name I can see my claim rule and information on why the claim is not supported in Azure AD. For this particular application, I also know that is better to use the AWS application from the Azure AD gallery, but it&#8217;s just an example to showcase the functionality.


![Image](./non-migratable-rule-was-detected-AWS-claim-1024x713.png)



### Stage 2 &#8211; Staged rollout (preview)  {.wp-block-heading}

The next step is to look into the magic of Staged rollout (preview). This feature allows us to test Azure AD Authentication using Groups so that we can stepwise safely move users. We can choose from Password Hash Sync or Pass-through Authentication. For the purpose of this post, I will only use PHS. 

staged-rollout-1024x517.png

First, we need to Go to our AD connect server and configure PHS. This can be found under the Optional Features. 


![image](./images/Password-Hash-Sync-enable-optional-AD-Connect.png)


The next step is to go back to the Staged Rollout feature to enable the functionality. 


![image](./Enable-staged-rollout-features-Preview.png)




In this post, I will choose only Password Hash Sync. You will get a prompt &#8220;Are you sure want to enable staged rollout for &#8216;Password Hash Sync


![image](./Enable-staged-rollout-features-Preview2.png)


NB: The maximum number of users in the initial configuration is 200 users. You can add additional users afterwards. For best results, validate in batches of 1,000 users. Dynamic and nested groups are not supported for staged rollout

I&#8217;m adding a Pilot group for my PHS users. <figure class="wp-block-image size-large is-resized">


![image](./Manage-groups-for-Password-Hash-Sync-AD-group-1-1024x252.png)


Now it&#8217;s time to test the user Authentication Flow from a browser. I&#8217;m using Fiddler to verify flow is working properly. The screenshots below shows that the flow is correct. 

![image](./Outlook-office365-login-phs-1024x425.png)

![image](./fiddler_PHS_login-1024x434.png)


Looking into Azure AD Audit Logs, we can see our object has status under Target and Modified Properties for User management and Hybrid Authentication that we have some new information. Our Object has to changed New Value PasswordHashSync. So with Audit logs, it&#8217;s easy to see which users has changed status. <figure class="wp-block-image size-large is-resized">


![image](./auditlogs-1024x406.png)

![image](./auditlogs2-1024x102.png)


### Summary

![image](./arch-1024x954.png)


Tools are of course not enough by itself to complete migration away from AD FS. However, the Application Insights and Staged Rollout are nice features when working with lager environments with multiple integrations, and need to verify that everything is working perfectly! I am pretty sure the Application Insight will improve more in Q1 2020, and it will be interesting to see features being improved. Using Staged Rollout to verify that our switch to PHS is working correctly is a welcome addition that I have been missing for years. Much better than big bang migrations! 

My earlier experience related to migration AD FS integrations has been in many cases just checking if the Application exists in the Azure AD gallery. Many Application also supports Automatic SCIM provisioning which is really great for governance control.  
If your application is not out-of-the-box try to schedule a workshop with the 3.party vendor and Developers, to setup a test environment to verify that you are able to integrate using Azure AD. In all my experiences this has not been any showstopper at all! In most cases, people just don&#8217;t know all the capabilities of Azure AD, and having a short meeting showing possibilities is enough to convince the organization for the switch.  
  
**I will try to update this article as these features go away from the preview.**  
  
Hopefully, this post can bring you some motivation to look into migration away from AD FS with Style 🙂 

**Recommended Links for further documentation Microsoft**
  
* https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/migrate-adfs-application-activity
  
* https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-staged-rollout

* https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/migrate-adfs-apps-to-azure

* https://portal.azure.com/#blade/Microsoft_Azure_ADHybridHealth/AadHealthMenuBlade/AdfsServicesList

* https://portal.azure.com/#blade/Microsoft_AAD_IAM/EnterpriseApplicationsInsightsMenuBlade/AppMigration