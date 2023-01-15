---
title: Windows Virtual Desktop with ARM and Azure DevOps
author: Stein-Erik Alvestad
date: 2020-10-20T15:36:12+00:00
url: /windows-virtual-desktop-with-arm-and-azure-devops/
categories:
  - Azure
  - Azure DevOps
  - WVD
  - Azure Virtual Desktop

---

### Introduction

In this post we will cover how to setup Windows Virtual Desktop ARM template with Azure DevOps. We walkthrough 2 different ARM templates. I&#8217;m using the ARM WVD templates from Microsoft: <a href="https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates" target="_blank" rel="noreferrer noopener">https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates</a> that came earlier in 2020. 

  * CreateAndProvisionHostPool
  * AddVirtualMachinesToHostPool

Figure 1: Visualized DevOps Workflow ARM pipelines 

![image](/wp-content/uploads/2020/10/image-29.png)


In Figure 1, we can see that the Azure DevOps build pipeline is integrated with Azure Key Vault for secrets management. We are using Service Connection with Azure, so that the ARM templates and PowerShell scripts can run in the context of the service principal. No Password are exposed in clear text to the pipeline!  
As part of the automation pipeline, one of my goals is to use Windows 10 image built using a Shared Image Gallery (SIG) built with Azure Image builder (AIB). Pointing to SIG is just part of the parameters file. I recommend using this guide to configure AIB and SIG:  
<a href="https://techcommunity.microsoft.com/t5/windows-virtual-desktop/building-a-windows-10-enterprise-multi-session-master-image-with/m-p/1503913" target="_blank" rel="noreferrer noopener">https://techcommunity.microsoft.com/t5/windows-virtual-desktop/building-a-windows-10-enterprise-multi-session-master-image-with/m-p/1503913</a>  
If you don&#8217;t have SIG ready, you can **optionally** just use the Windows 10 SKU from Gallery if you just need a quick POC. Just change the parameters  
  
I&#8217;ve added the &#8220;**arm-wvd-CreateAndProvisionHostPool.json**&#8221; and arm-wvd- to my GitHub repo, this will be used by the pipeline. ****  
**NB!** You will need update the parameters in the &#8220;**CreateAndProvisionHostPool.parameters.json&#8221;** yourself. The parameters must be equal to your own environment! I will go deeper into some of the parameters used by the arm template. There are also some PowerShell scripts that the pipelines will use.  
I&#8217;m by no means any expert in ARM or PowerShell, but I will cover the basics of getting these pipelines working.

The Templates and code used is available from my GitHub repo. Clone the repo, so that you can use the files from <a href="https://github.com/alventech/wvd-arm-devops" target="_blank" rel="noreferrer noopener">https://github.com/alventech/wvd-arm-devops</a>  
  
**CreateAndProvisionHostPool**

![image](/wp-content/uploads/2020/10/image-31-1024x387.png)

**AddVirtualMachinesToHostPool**<figure class="wp-block-image size-large is-resized">

![image](/wp-content/uploads/2020/10/image-41-1024x381.png)

### <span class="ez-toc-section" id="Prerequisites"></span>Prerequisites<span class="ez-toc-section-end"></span> {.wp-block-heading}

  * Azure Subscription
  * VNET and Subnet
  * Azure Key Vault 
  * Azure DevOps Project
  * Active Directory Domain or Azure Domain Services to join
  * Shared Image Gallery (Windows 10 Image)
  * GitHub or any other source control that you prefer. 

### Create And ProvisionHostPool  

**Azure DevOps Project**

Let&#8217; start by getting our Azure DevOps Project ready for the CreateAndProvisionHostPool.  
  
Go to <a href="https://dev.azure.com/" target="_blank" rel="noreferrer noopener">https://dev.azure.com/</a> and create your first project  
Choose Project Settings to the bottom left.

![image](/wp-content/uploads/2020/10/image-11.png)

### Service Connections

![image](/wp-content/uploads/2020/10/image-12.png)

Add service connection for your GitHub account and Azure Subscription.

![image](/wp-content/uploads/2020/10/image-5-1024x339.png)


Go back to the Pipeline tab on the left Side and choose **New release pipeline.** 

![image](/wp-content/uploads/2020/10/image.png)

Create an **Empty Job**

![image](/wp-content/uploads/2020/10/image-1.png )

Name your Release pipeline, example &#8220;**wvd.arm.create.hostpool**&#8220;, and hit save.

![image](/wp-content/uploads/2020/10/image-2.png)

### Artifact

From the **artifacts**, add your GitHub account, or the source control of your choice. Click save When You&#8217;re Done. 

![image](/wp-content/uploads/2020/10/image-6-1024x530.png)

![image](/wp-content/uploads/2020/10/image-7-255x300.png)


### Variable Groups  

Go to the Pipelines Tab &#8211; **Library** and choose the Variable Groups below. 

![image](/wp-content/uploads/2020/10/image-13.png)

Create a **New Variable Group**

![image](/wp-content/uploads/2020/10/image-14-1024x635.png)


Authorize the **Key vault** to your subscription and key vault name.

![image](/wp-content/uploads/2020/10/image-15-845x1024.png)


When Authorized to the **key Vault**, remember to add your secret name Variable. This used by the ARM template parameters.

![image](/wp-content/uploads/2020/10/image-17.png)


### Paramters-CreateAndProvisionHostPool

The ARM parameters values in arm-wvd-**CreateAndProvisionHostPool.paramters.json** match your environment. Below I&#8217;ve just listed all the parameters that simply got from the Azure Portal, so that I could see what parameters are generated the parameter.  
  
![image](/wp-content/uploads/2020/10/image-42.png
)

I Downloaded the Template and used the parameter file generated to see how the syntax for each parameter needed to be. 

![image](/wp-content/uploads/2020/10/image-43.png)


I will cover the parameters that is not obvious. So that I could more seamlessly see what parameters needs to be added. You can also &#8220;get-AzGalleryImageDefinition&#8221;. If we Look more into the Azure Share Image Gallery (SIG) parameters, All the values, was then added to the values in &#8220;**vmTemplate**&#8221; parameter.  
The &#8220;**customImageId**&#8221; SIG can be found in the azure portal as well.

![image](/wp-content/uploads/2020/10/image-9-1024x500.png)

With the Parameter &#8220;**vmCustomImageSourceId**&#8220;, we also need to add SIG resoruce ID to this value

#### PowerShell Task ExpirationTime

When we have all the parameter&#8217;s updated, we ready to add the PowerShell task. The Task is used to generate the parameter value for &#8220;**tokenExpirationTime**&#8220;. The PowerShell task will store the variable to the next DevOps ARM Task. The Script will run &#8220;**get-date**&#8221; with correct format. The write-host will bring the variable to the next task. 

![image](/wp-content/uploads/2020/10/image-18-1024x342.png)


Search for **PowerShell**, and add the task. 

![image](/wp-content/uploads/2020/10/image-19-1024x205.png)

Choose File Path &#8211; Create the Display Name and choose Script Path to the **&#8220;createtokenExpirationTime.ps1&#8221;** script.  
  
It&#8217;s located in the <a href="https://github.com/alventech/wvd-arm-devops/tree/main/arm-wvd-CreateAndProvisionHostPool" target="_blank" rel="noreferrer noopener">https://github.com/alventech/wvd-arm-devops/tree/main/arm-wvd-CreateAndProvisionHostPool

![image](/wp-content/uploads/2020/10/image-20-1024x340.png)

Go to Tasks &#8211; add tasks &#8211; search for **ARM,** add the **ARM template deployment** as a task.

![image](/wp-content/uploads/2020/10/image-21-1024x184.png)

**Resource group** and **location** must be equal to your environment. 

![image](/wp-content/uploads/2020/10/image-22.png)

In the screenshot below you can see more of the ARM settings.  
**Template** &#8211; Change the location to our GitHub artifact for &#8220;**arm-wvd-CreateAndProvisionHostPool.json**&#8221; template location.  
**Template parameters** &#8211; Change the location to GitHub artifact for &#8220;**arm-wvd-CreateAndProvisionHostPool.parameters.json**&#8221; Parameters.  
The templates are located in <a href="https://github.com/alventech/wvd-arm-devops/tree/main/arm-wvd-CreateAndProvisionHostPool" target="_blank" rel="noreferrer noopener">https://github.com/alventech/wvd-arm-devops/tree/main/arm-wvd-CreateAndProvisionHostPool</a>  
  
In the **Override template parameters**, we will set  
&#8220;**administratorAccountPassword**&#8221; variable and the &#8220;**tokenExperationTime**&#8221; is the variable from the PowerShell task we created earlier.  
  
-administratorAccountPassword &#8220;$(wvd-ad-join)&#8221; -tokenExpirationTime &#8220;$(ExpirationTime)&#8221;  

![image](/wp-content/uploads/2020/10/image-23.png
)

### Pipeline_Variables_-_override_even_more_parameters

if you don&#8217;t like working with the &#8220;**CreateAndProvisionHostPool.paramters.json**&#8220;, you can just override even more parameters for example &#8220;**ouPath**&#8221; and &#8220;**administratorAccountUsername**&#8221; that you down want to be coded in the parameter json. I will recommend choosing Variables &#8211; Pipeline Variables and variables to match your needs.

![image](/wp-content/uploads/2020/10/image-28-1024x251.png)

you **must** remember to override it the arm template as well! 

![image](/wp-content/uploads/2020/10/image-30-1024x115.png)

### Create a new release and Deploy

If you&#8217;re done with updating all the parameters, we should be ready for a release. **Create release** and **deploy.** 

![image](/wp-content/uploads/2020/10/image-25-1024x404.png)


In the logs you can see that the Agent has finalized the job. Go to the troubleshooting guide at the end if you get any problems. 

![image](/wp-content/uploads/2020/10/image-26-1024x484.png)

### Pipeline Add Virtual Machines To HostPool

When the ARM template deployment is finished, we can go to the next Pipeline &#8211; **AddVirtualMachinesToHostPool**. This&#8217;s probably the one, we&#8217;ll use more often for your Maintenace. To get started we must create a new release pipeline, example **&#8220;wvd.arm.AddVMToHostPool**&#8220;. Add you Artifact and Stage and click save. 

![image](/wp-content/uploads/2020/10/image-35.png)

#### PowerShell Task RdsRegistration Info token

In this Task we will configure a PowerShell task with Azure Powershell.  
We will use the https://github.com/alventech/wvd-arm-devops/blob/main/arm-wvd-  
AddVirtualMachinesToHostPool/RdsRegistrationInfotoken.ps1 scirpt. I&#8217;ve borrowed the script from <a href="https://twitter.com/fberson" target="_blank" rel="noreferrer noopener">https://twitter.com/fberson</a>, and added some small modifications. The main goal for the scirpt is to obtain the **RdsRegistrationInfotoken&nbsp;**and send the variable to the next task for the ARM&nbsp;parameter&nbsp;**hostpoolToken.**  
This will ensure that we have a valid token, or generate a new if token has expired. A Token is required from the Host Pool to allow, new VM&#8217;s to a Host Pool. The write-host will bring the variable to the next task.  
  
  
To get stated I will add the &#8220;Azure PowerShell&#8221; 

![image](/wp-content/uploads/2020/10/image-36.png)

Before running the pipeline and the script, you will need to update the variables to your own environment. _**Optionally**_, you could just use an Inline script a Copy Past scirpt, if you prefer that. 

$azureSubscriptionID&nbsp;=&nbsp;&#8220;your-wvd-sub-id&#8221;  
$resourceGroupName&nbsp;=&nbsp;&#8220;rg-wvd-Pooled-desktop&#8221;  
$existingWVDHostPoolName&nbsp;=&nbsp;&#8220;HostPool-Test&#8221;

Next go to the **Authenticate** to your Subscription to allow the task to use Service Connection and Service Principal, that will authenticate to PowerShell during the task. 

![image](/wp-content/uploads/2020/10/image-37.png)

Add your **Template** and **Template parameters**, and **Override template parameters** 

-administratorAccountPassword &#8220;$(wvd-ad-join)&#8221; -hostpoolToken &#8220;$(RdsRegistrationInfotoken)&#8221;



### ARM_Template_Deployment_Task_-_AddVmsToHoostPool

![image](/wp-content/uploads/2020/11/image-1024x219.png)

### Link_Variable_group_to_pipeline

The last part of the pipeline is to link the variable group, so that pipeline can use the Key Vault credentials authorized to the DevOps Project earlier. 

![image](/wp-content/uploads/2020/10/image-39-1024x293.png)

### Create_a_new_release_and_Deploy-2"></span>Create a new release and Deploy

![image](/wp-content/uploads/2020/10/image-40-1024x343.png)

You could also automate the drain Mode for your sessions host.  
Like example <a href="https://www.nielskok.tech/windows-virtual-desktop/wvd-set-drain-mode-powershell/" target="_blank" rel="noreferrer noopener">https://www.nielskok.tech/windows-virtual-desktop/wvd-set-drain-mode-powershell/</a> in the pipeline. I prefer to schedule, this at a later stage since, I like to know that everything is working as it should with the new image.

### Troubleshooting

During this project I&#8217;ve failed multiple times because of parameter or value that was in the wrong context. Your DevOps pipeline will give you some details about the error, but it&#8217;s not always easy to see what failed for ARM, so I prefer looking into the **RAW Error** from the Resource Group and **Deployments** tab. I really recommend investing some time in this ARM video series from <a href="https://www.youtube.com/c/SamCogan/" target="_blank" rel="noreferrer noopener">https://www.youtube.com/c/SamCogan/</a> if you need some advice regarding test and validation in arm templates. It&#8217;s for sure helped me a lot. In the picture below the task failed because I was using the wrong prefix. Looking into the json file I see that the variable was using rdshPrefix to the vmNamePrefix.

![image](/wp-content/uploads/2020/10/image-33-1024x368.png)


Another example the passsword that did not work. A closer look to the varaible group for the pipeline, and I could see that the correct variable group was not linked. 

![image](/wp-content/uploads/2020/10/image-34.png)

Spaces in DevOps task Varaible with * https://gist.github.com/alventech/a80d1c5be5cabfef30e46cef27f6754c createtokenExpirationTime.ps1

**Update** 02.06.2021 (thnx to one of my readers Ben! We found that I had a bug with spaces in the Gist Github file!) this file has been updated so that the variable with ExpirationTime will be correct ). If you copied the code prior to 02.06.2021 the variable would cause a space to be generated which could cause some issues with the parameter input.  
I&#8217;ve also updated the task.setvariable that was wrong in github and gist github.



### Summary

It was a nice learning curve to configure WVD ARM templates with DevOps. What I like about Azure DevOps with the ARM templates, is that it gives additional security, and flexibility and scalability when you need to do deploy across multiple environments. You can export your templates, so it&#8217;s more repeatable in your CI/CD environments. With the override parameters is quick and easy to change parameters at your needs. 

### <span class="ez-toc-section" id="Documentation"></span>**Documentation:** <span class="ez-toc-section-end"></span> {.wp-block-heading}

<a href="https://docs.microsoft.com/en-us/powershell/module/az.desktopvirtualization/new-azwvdhostpool?view=azps-4.8.0" target="_blank" rel="noreferrer noopener">https://docs.microsoft.com/en-us/powershell/module/az.desktopvirtualization/new-azwvdhostpool?view=azps-4.8.0</a>

<https://www.christiaanbrinkhoff.com/2020/05/01/windows-virtual-desktop-technical-2020-spring-update-arm-based-model-deployment-walkthrough/>

<https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates>