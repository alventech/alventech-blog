---
weigth: 1
title: Gearing up for Azure AD Connect cloud provisioning
author: Stein-Erik Alvestad
type: post
date: 2020-01-07T21:18:23+00:00
url: /gearing-up-for-azure-ad-connect-cloud-provisioning/
categories:
  - AAD
  - AD Connect

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: ""

toc:
  auto: false

---

{{< figure src="/images/AzureAD_Start_gearingup-1024x768.png" title="Lighthouse (figure)" >}}


![image](/AzureAD_Start_gearingup-1024x768.png)


In this post, we will cover the **Azure AD Connect cloud provisioning** (**preview)** released in November 2019. The **Cloud provisioning** is an Agent that can bridge between on-prem and Azure AD to sync users. I don&#8217;t see cloud provisioning as a replacement for AD Connect, but more like simplifying the way to configure user provisioning, across more complex setups with multiple or disconnected forests. Trying to achieve this earlier with AD Connect, requires a complex setup dependent on AD Trust. The **Azure AD Connect cloud provisioning** is much similar functionality Okta has had for some time, so my best guess is that Microsoft move is to gap the missing functionality. The tool will most likely get more features throughout 2020 🙂  
  
My goal with this post is, to cover the use case of the **Azure AD Connect cloud provisioning** with an existing forest and a new forest with a Single Azure AD Tenant. Below is a high-level design, this shows my current setup. To the left, we have my domain that uses AD Connect and the disconnected forests to the right, which uses Azure AD Connect cloud provisioning. Both will sync to the single Azure AD Tenant. There is no site-to-site VPN or any Active Directory Trust between these Forests. 

![image](hld_AADCP_v1-1024x790.png)

![image](/static/testimages/01/fido2-boba-feitian.png)

### Step 1 &#8211; Enable **Azure AD Connect cloud provisioning**  {.wp-block-heading}

The first step to enable Azure AD Connect cloud provisioning is getting the agent from Azure AD 

* https://portal.azure.com/#blade/Microsoft\_AAD\_IAM/ActiveDirectoryMenuBlade/AzureADConnect][1]  
<figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/CloudProvosing-1024x604.png" alt="" class="wp-image-814" width="768" height="453" srcset="/wp-content/uploads/2020/01/CloudProvosing-1024x604.png 1024w, /wp-content/uploads/2020/01/CloudProvosing-300x177.png 300w, /wp-content/uploads/2020/01/CloudProvosing-768x453.png 768w, /wp-content/uploads/2020/01/CloudProvosing-1060x626.png 1060w, /wp-content/uploads/2020/01/CloudProvosing-550x325.png 550w, /wp-content/uploads/2020/01/CloudProvosing-847x500.png 847w, /wp-content/uploads/2020/01/CloudProvosing.png 1091w" sizes="(max-width: 768px) 100vw, 768px" /> </figure> 

Choose &#8220;**Manage provisioning**&#8221; in the Azure AD Connect cloud provisioning. 

Go to Download Agents, to download  
&#8220;**AADConnectProvisioningAgentSetup.exe** &#8220;agent.  
In terms of HA with cloud provisioning, just install more agents split on other servers per forest. <figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/Donwload-Agent-1024x261.png" alt="" class="wp-image-818" width="768" height="196" srcset="/wp-content/uploads/2020/01/Donwload-Agent-1024x261.png 1024w, /wp-content/uploads/2020/01/Donwload-Agent-300x76.png 300w, /wp-content/uploads/2020/01/Donwload-Agent-768x196.png 768w, /wp-content/uploads/2020/01/Donwload-Agent-1060x270.png 1060w, /wp-content/uploads/2020/01/Donwload-Agent-550x140.png 550w, /wp-content/uploads/2020/01/Donwload-Agent.png 1213w" sizes="(max-width: 768px) 100vw, 768px" /> </figure> 

Run the &#8220;**AADConnectProvisioningAgentSetup.exe**&#8221; to complete the setup. The agent requires authentication against Azure AD. <figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/agent-install2-1024x251.png" alt="" class="wp-image-839" width="768" height="188" srcset="/wp-content/uploads/2020/01/agent-install2-1024x251.png 1024w, /wp-content/uploads/2020/01/agent-install2-300x74.png 300w, /wp-content/uploads/2020/01/agent-install2-768x188.png 768w, /wp-content/uploads/2020/01/agent-install2-1060x260.png 1060w, /wp-content/uploads/2020/01/agent-install2-550x135.png 550w, /wp-content/uploads/2020/01/agent-install2.png 1092w" sizes="(max-width: 768px) 100vw, 768px" /> </figure> 

When adding privileges to the Domain, use a service account that does not have password change requirement, because changing the password will break the agent. This will require the agent to be reconfigured for the Add the Active Directory, with domain admin privileges. <figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/agent-install4-1024x720.png" alt="" class="wp-image-840" width="768" height="540" srcset="/wp-content/uploads/2020/01/agent-install4-1024x720.png 1024w, /wp-content/uploads/2020/01/agent-install4-300x211.png 300w, /wp-content/uploads/2020/01/agent-install4-768x540.png 768w, /wp-content/uploads/2020/01/agent-install4-1060x745.png 1060w, /wp-content/uploads/2020/01/agent-install4-550x387.png 550w, /wp-content/uploads/2020/01/agent-install4-711x500.png 711w, /wp-content/uploads/2020/01/agent-install4.png 1101w" sizes="(max-width: 768px) 100vw, 768px" /> </figure> <figure class="wp-block-image size-large"><img decoding="async" loading="lazy" width="1024" height="715" src="/wp-content/uploads/2020/01/agent-install6-2-1024x715.png" alt="" class="wp-image-869" srcset="/wp-content/uploads/2020/01/agent-install6-2-1024x715.png 1024w, /wp-content/uploads/2020/01/agent-install6-2-300x209.png 300w, /wp-content/uploads/2020/01/agent-install6-2-768x536.png 768w, /wp-content/uploads/2020/01/agent-install6-2-1060x740.png 1060w, /wp-content/uploads/2020/01/agent-install6-2-550x384.png 550w, /wp-content/uploads/2020/01/agent-install6-2-716x500.png 716w, /wp-content/uploads/2020/01/agent-install6-2.png 1096w" sizes="(max-width: 1024px) 100vw, 1024px" /></figure> 

After the installation, the service will run 2 services **Microsoft Azure AD Connect Agent Updater** and **Microsoft Azure AD Connect Provisioning Agent**. There nothing much we can change after the agent has been installed.<figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-1024x131.png" alt="" class="wp-image-844" width="768" height="98" srcset="/wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-1024x131.png 1024w, /wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-300x38.png 300w, /wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-768x98.png 768w, /wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-1536x196.png 1536w, /wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-2048x261.png 2048w, /wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-1060x135.png 1060w, /wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-550x70.png 550w, /wp-content/uploads/2020/01/services_agents_CP_ADConnect-1-1920x245.png 1920w" sizes="(max-width: 768px) 100vw, 768px" /> </figure> 

We can go back to Azure AD and configure the agent.  
[https://portal.azure.com/#blade/Microsoft\_AAD\_IAM/ActiveDirectoryMenuBlade/AzureADConnect][1] &#8211; Choose Manage Provisioning. <figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/Azure-AD-Provisioning-Preview1-Azure-Active-Directory.png" alt="" class="wp-image-845" width="668" height="605" srcset="/wp-content/uploads/2020/01/Azure-AD-Provisioning-Preview1-Azure-Active-Directory.png 891w, /wp-content/uploads/2020/01/Azure-AD-Provisioning-Preview1-Azure-Active-Directory-300x272.png 300w, /wp-content/uploads/2020/01/Azure-AD-Provisioning-Preview1-Azure-Active-Directory-768x696.png 768w, /wp-content/uploads/2020/01/Azure-AD-Provisioning-Preview1-Azure-Active-Directory-550x498.png 550w, /wp-content/uploads/2020/01/Azure-AD-Provisioning-Preview1-Azure-Active-Directory-552x500.png 552w" sizes="(max-width: 668px) 100vw, 668px" /> </figure> 

It has automaticly popluated the domain field from the Agent I installed.  
The Scope Users give us 3 opportunities. All Users, Selected Security Group and Selected Organizational Units. I&#8217;m using the OU based on this demo environment. you can add multiple OU&#8217;s, but not combine with example OU and Security groups form my testing. <figure class="wp-block-image size-large">

<img decoding="async" loading="lazy" width="632" height="230" src="/wp-content/uploads/2020/01/OUDN_socpe_users_CP_ADconnect.png" alt="" class="wp-image-848" srcset="/wp-content/uploads/2020/01/OUDN_socpe_users_CP_ADconnect.png 632w, /wp-content/uploads/2020/01/OUDN_socpe_users_CP_ADconnect-300x109.png 300w, /wp-content/uploads/2020/01/OUDN_socpe_users_CP_ADconnect-550x200.png 550w" sizes="(max-width: 632px) 100vw, 632px" /> </figure> <figure class="wp-block-image size-large"><img decoding="async" loading="lazy" width="609" height="247" src="/wp-content/uploads/2020/01/multipleOus-1.png" alt="" class="wp-image-850" srcset="/wp-content/uploads/2020/01/multipleOus-1.png 609w, /wp-content/uploads/2020/01/multipleOus-1-300x122.png 300w, /wp-content/uploads/2020/01/multipleOus-1-550x223.png 550w" sizes="(max-width: 609px) 100vw, 609px" /></figure> 

The last part at step 4 Deploy, can choose the enable switch.  
The provisioning configuration to users and groups.  
<figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/enableCPADconnect.png" alt="" class="wp-image-851" width="649" height="435" srcset="/wp-content/uploads/2020/01/enableCPADconnect.png 865w, /wp-content/uploads/2020/01/enableCPADconnect-300x201.png 300w, /wp-content/uploads/2020/01/enableCPADconnect-768x515.png 768w, /wp-content/uploads/2020/01/enableCPADconnect-550x369.png 550w, /wp-content/uploads/2020/01/enableCPADconnect-746x500.png 746w" sizes="(max-width: 649px) 100vw, 649px" /> </figure> 

After saving we should see our configuration in the console. <figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/healthyAgent-1024x248.png" alt="" class="wp-image-852" width="768" height="186" srcset="/wp-content/uploads/2020/01/healthyAgent-1024x248.png 1024w, /wp-content/uploads/2020/01/healthyAgent-300x73.png 300w, /wp-content/uploads/2020/01/healthyAgent-768x186.png 768w, /wp-content/uploads/2020/01/healthyAgent-1060x257.png 1060w, /wp-content/uploads/2020/01/healthyAgent-550x133.png 550w, /wp-content/uploads/2020/01/healthyAgent.png 1101w" sizes="(max-width: 768px) 100vw, 768px" /> </figure> 

The test users synced pretty instant.  
  
Looking into the user attributes with Powershell, we see the details about on-prem AD details. 

<div class="wp-block-group is-layout-flow">
  <div class="wp-block-group__inner-container">
    <pre class="wp-block-code"><code>Get-AzureADUser -ObjectId $UserId | Select -ExpandProperty ExtensionProperty</code></pre>
  </div>
</div><figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/powershell_CP_user.png" alt="" class="wp-image-860" width="905" height="137" srcset="/wp-content/uploads/2020/01/powershell_CP_user.png 905w, /wp-content/uploads/2020/01/powershell_CP_user-300x45.png 300w, /wp-content/uploads/2020/01/powershell_CP_user-768x116.png 768w, /wp-content/uploads/2020/01/powershell_CP_user-550x83.png 550w" sizes="(max-width: 905px) 100vw, 905px" /> </figure> 

Verifying Azure AD signing with PHS works aswell. <figure class="wp-block-image size-large is-resized">

<img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/logintest1.png" alt="" class="wp-image-859" width="438" height="310" srcset="/wp-content/uploads/2020/01/logintest1.png 584w, /wp-content/uploads/2020/01/logintest1-300x212.png 300w, /wp-content/uploads/2020/01/logintest1-550x389.png 550w" sizes="(max-width: 438px) 100vw, 438px" /> </figure> <figure class="wp-block-image size-large is-resized"><img decoding="async" loading="lazy" src="/wp-content/uploads/2020/01/logintest2-1024x292.png" alt="" class="wp-image-862" width="768" height="219" srcset="/wp-content/uploads/2020/01/logintest2-1024x292.png 1024w, /wp-content/uploads/2020/01/logintest2-300x85.png 300w, /wp-content/uploads/2020/01/logintest2-768x219.png 768w, /wp-content/uploads/2020/01/logintest2-1060x302.png 1060w, /wp-content/uploads/2020/01/logintest2-550x157.png 550w, /wp-content/uploads/2020/01/logintest2.png 1376w" sizes="(max-width: 768px) 100vw, 768px" /></figure> 

### Summary {.wp-block-heading}

The Azure AD Connect cloud provisioning feature is a step in the right direction for Microsoft to fill the missing gap against their competitors, and the use case is absolutely something I will consider for disconnected on-premises Environments. But this will depend highly on what functionality is required and if the environment is not to complex.  
  
The table from Microsoft documentation <https://docs.microsoft.com/en-us/azure/active-directory/cloud-provisioning/what-is-cloud-provisioning?> is great to compare features between **Azure Active Directory Connect sync** vs **Azure Active Directory Connect cloud provisioning**.  
The comparison will help to determine which use case you could fit.  
  
**I will try to update this post as these features go away from the preview.**  


Recommended Links for further documentation from Microsoft : 

<https://docs.microsoft.com/en-us/azure/active-directory/cloud-provisioning/tutorial-single-forest>  
  
<https://docs.microsoft.com/en-us/azure/active-directory/cloud-provisioning/tutorial-existing-forest>

 [1]: https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect