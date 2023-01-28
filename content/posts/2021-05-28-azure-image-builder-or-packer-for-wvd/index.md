---
weigth: 1
title: Azure Image Builder or Packer for Azure Virtual Desktop
author: Stein-Erik Alvestad
date: 2021-05-28T19:08:12+00:00
url: /azure-image-builder-or-packer-for-wvd/
categories:
  - Azure DevOps
  - Azure Image Builder
  - Azure Virtual Desktop
  - packer
  - WVD

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: "start_bob_builder.png"

toc:
  auto: false
---


### Introduction

So you&#8217;re in the situation seeking to automate Azure Virtual Desktop (AVD) image creation. You have discovered Packer and Azure Image Builder, but do not know which to use. Hopefully, I will as short as possible try to make sense of the differences, and why you fancy one of them depending on your scenario. Lucky for you it&#8217;s no wrong answer, the secret is that you use packer either way 🙂 

In the table below I&#8217;ve listed some important key capabilities from both services.  
To read the entire specification I will also recommend reading the **documentation** section below that I&#8217;ve added in this post. 

Concerning the initial table, I&#8217;ve additionally explained how you can automate AIB and packer (azure-arm) with Azure DevOps using the marketplace extensions. You do not need to use these, you could also just interact with Azure CLI, etc. My goal is just to show a quick proof of concept with Windows 10 multisession image. Both examples distribute images to SIG.  

|Description |Packer (azure-arm)|AIB (azure image builder)|
|-----|--------|---|
|Managed by|HashiCorp       |"Microsoft" ( with HasiCorp Packer under the hood) |
|Azure RBAC role  | Service Principal | Managed Identity |
| Azure Resource Group |   Provision resources to the default resource group from packer or existing resource group |  Provision resources to the default resource group from packer or existing resource group                 |
| Distribute           | Shared Image GalleryManagedImage,VHD             |       Shared Image GalleryManagedImage,VHD           |
| Communicators (windows) - Packer create certificate only valid 24-hours from invocation time. Uploaded to KeyVault |  winrm            | winrm                 |
|  Provisioner          |    Powershell, chef, ansible, puppet , DSC          |    Powershell              |
| Region                | All Azure regions |  https://learn.microsoft.com/en-us/azure/virtual-machines/image-builder-overview?tabs=azure-powershell#regions    | 
| Hyper-V Generation (images) https://docs.microsoft.com/en-us/azure/virtual-machines/generation-2  | gen1 and gen 2 |  gen1 and gen 2 (roadmap)  |
| Azure DevOps / Infra as Code | Packer task / Build Image task (1.3.4) / Azure CLI and Powershell (1.7.*) | Azure VM Image Builder Task (preview), Azure CLI and Powershell (1.7.*) |
| Security  | VNET (default or existing) , NSG (restricted ports) , KeyVault (certificate and secrets) | VNET (default or existing) , NSG (restricted ports), KeyVault (certificate and secrets) |
| Packer.exe | Need to download the latest packer version or specify an older version | Do not have any direct interaction with Packer. Everything goes through the azure image builder service that again uses packer under the hood |
| Configuration syntax | JSON or HCL (preferred from 1.7.0) | JSON |
| Logging | Packer logs are stored in storageaccount | Packer logs are stored in storageaccount |





_20.02.2021 the Azure Image builder Service was announced Generally available, but Microsoft suddenly apologized to the publication and pulled the GA status. I do not think it far away, and we could probably still expect it to hit GA Q2 or Q3 in 2021._  


>

**Update**  
07.06.2021 _Azure Image builder Service was announced Generally available_.

>

 
### Image Creation Deployment High Level Design

If we look into the high-level design of how we can create and deploy WVD, it could look similar to this process. The building blocks for WVD image creation and image deployment will involve many phases before we have the end product. With help from packer, AIB ,SIG Azure DevOps we have tools to support automation of this process. 


![image](/wp-content/uploads/2021/05/wvd_imgbuildandeployment-2-1024x684.png)



  1. use marketplace SKU (add this to our code)
  2. choose the latest offer, windows version supported by your organization (add this to code)
  3. build pipeline with AIB or Packer with info from 1 & 2
  4. provisioner &#8211; install windows features and custom applications from azure storage/azure files /package manager
  5. distribute the shared image gallery across regions
  6. trigger ARM or TF deployment with the latest release of the ID from SIG. 
  7. start windows customization extensions phase
  8. domain join extension
  9. host pool join dsc extension
 10. custom PowerShell extension (example WVD optimization script )
 11. VM added to host pool ready for testing or production. 

### Getting started with Azure Image builder with Azure DevOps

Following that the AIB is not GA, we are required to register the preview service. Follow this guide from Microsoft to **VirtualMachineTemplatePreview** for the preview. <https://docs.microsoft.com/en-us/azure/virtual-machines/windows/image-builder-powershell#register-features> . _**This step is not required when GA!**_ 

I&#8217;ve already created Managed Identity with permissions and added permissions to allow SIG with the guide below.

<https://docs.microsoft.com/en-us/azure/virtual-machines/windows/image-builder-powershell#create-user-identity-and-set-role-permissions>

When all prerequisites are in place we can start adding the Azure VM Image Builder Test (Preview) in Azure DevOps. You can also use azure cli or powershell task´s. 

### Create Azure DevOps Project

The first task is to create a new project in Azure DevOps dev.azure.com. I will do it manually, but you can do this with Terraform if you need to automate this multiple times: <https://techcommunity.microsoft.com/t5/itops-talk-blog/how-to-use-terraform-to-create-azure-devops-projects/ba-p/1555471>. We can use the same project for both pipelines. 

### Add the marketplace task and install the Marketplace extension for AIB preview


You can grab the Marketplace extension for AIB (preview) from <https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder>

### Add you github service connection

I&#8217;m using Github as source control. 

### Create your pipeline for Azure Image Builder

Choose pipelines &#8211; Classic editor and connect your github account and empty job 

![image](/wp-content/uploads/2021/05/pipelines.png)

I&#8217;m using the classic editor to create the pipeline. 

![image](/wp-content/uploads/2021/05/pipelines-classic-editor.png

go back to the release pipeline and add your Azure Image Builder tas

![image](/wp-content/uploads/2021/04/add_ADO_task-1024x437.png
)

Let&#8217;s look into the required fields for the extension.  
In the first step of the template, we need to Add your azure subscription, resource, and managed identity resource id.  

![image](/wp-content/uploads/2021/05/aib_task_1.png)

In the Customize step set your custom script. It will copy the scirpt from my github workingdirectory and copy it over to the directory on the Virtual Machine. The default location for Inline customization script will runt from the VM in &#8220;C:\BuildArticacts&#8221; in this step you can install apps or do any custom configuration that you need. I  
It&#8217;s not any requirement to specify Sysprep. AIB will do the Sysprep for us!

![image](/wp-content/uploads/2021/05/aib_task_2.png)

**NB!** I&#8217;ve specified only WestEurope as SIG region, If you choose more than 1, your build will fail because it exceeded the 1-hour limit.  
(One free job that can run for up to 60 minutes each time). It does help much to choose a bigger VM because SIG is the slowest part. Even though the job fails the SIG will succeed after checking the Azure portal.  
  
You can buy and read more about parallel jobs :

* https://docs.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops&tabs=ms-hosted"

* https://docs.microsoft.com/en-us/azure/devops/pipelines/licensing/concurrent-jobs?view=azure-devops&tabs=ms-hosted  

![image](/wp-content/uploads/2021/05/aib_task_3.png)


Running the build will output this on successful task. 

![image](/wp-content/uploads/2021/05/aib_task_run_status-1-1024x848.png)


You can also check Shared Image Gallery to verify your newly added image from the AIB Pipeline.

![image](/wp-content/uploads/2021/05/aibdef_SIG-1-1024x607.png)

The final YAML pipeline will look like these. You can use this to standardize your azure pipeline deployments for CI /CD

### Getting started with Packer azure-arm with Azure DevOps

  
Many other blogs use the **Build machine image** task to build an immutable image. My problem with the task is that it uses packer version 1.3.4, which is old. I want to use the latest packer version because I want to use the Shared Image Gallery with packer. So to solve this we use Azure CLI or Powershell that use our packer JSON or HCL file, which will be used by the pipeline as input. With Azure CLI or Powershell you should from my experiance always get the latest packer version installed. I will use the JSON template since I&#8217;ve not upgraded my templates to HCL. (upgrading to HCL will be another blog). 

For authentication, I will use a service principle. _NB!_ If you want to use a Managed Identity, you need to create an Azure Virtual Machine that has Managed Identity enabled, and then use this VM to run your build. 

### Variable group

The first step is to add Variable Group. All variables that Packer needs are stored here.

![image](/wp-content/uploads/2021/05/var_group_1.png)

In My KeyVault I&#8217;ve added all details from my Service Principal account that Packer will use for (azure-arm). 

![image](/wp-content/uploads/2021/05/KeyVault_1.png)

I&#8217;ve added all secrets to Key Vault &#8220;wvd-kv&#8221; and all other variables to &#8220;wvd-packer-vars&#8221; as shown in the picture below.  
To link the Variable group to the DevOps project choose variables group add key vault.  
Remember to link secrets as key vault variables.

![image](/wp-content/uploads/2021/05/var_group_2.png)

### Create your Azure DevOps Pipeline for Packer

Let&#8217;s configure the Pipeline. I&#8217;ve added 2 tasks. Packer validate and Packer build

choose pipelines &#8211; Classic editor and connect your github account and empty job as we did in the steps above earlier. 

![image](/wp-content/uploads/2021/05/packer_devops_task_overview.png)

![image](/wp-content/uploads/2021/05/packer_validate_task.png)

In the Inline Script I will add the validate shell script. To verify that there are no obvious parameters missing.

![image](/wp-content/uploads/2021/05/packer_build_task.png)

In the Inline Script I will add the build shell script. I&#8217;m adding the variables with export and generating timestamp for one of the variables (image_version) to set the semantic version that&#8217;s required by SIG.

The Packer template will in summary use variables we have defined. Create Image with windows 10, sysprep VM add it to SIG.

To fully understand user variables go to https://www.packer.io/docs/templates/legacy\_json\_templates/user-variables

Link Variable Groups to the Packer pipeline. This is important step. Without adding the Variables the build will fail! 

![image](/wp-content/uploads/2021/05/link_VG_packer-1024x419.png)

When running the build we can see that it&#8217;s started the azure-arm packer Build stage. 

![image](/wp-content/uploads/2021/05/run_build_logs-1024x496.png)

In Shared Image Gallery we can find the new image created by the Packer build template. 

![image](/wp-content/uploads/2021/05/pakcerdef_SIG-1-1024x415.png)

The YAML pipeline for Packer for setups looks like this. 

You can use this to standardize your azure pipeline deployments for CI /CD 



### Troubleshooting

**Packer**:  
if you see this error with Packer below. You have most likely not configured or linked the variable group or added the wrong variable for Client Secret, ClientID etc .

azure-arm: Running builder  
azure-arm: Getting tokens using Managed Identity for Azure  
  
**AIB**:  
if you see this error with AIB below. You have not given correct access to your managed identity for either shared image gallery or storage account.  
  
[error]Error: put template call failed for template t\_undefinedundefined\_1622201565931 with error: Not authorized to access the resource: /subscriptions.  
  
If your packer stops under the deployment all logs are stored in the storage account. Use **CMTrace**.exe to debug the log file.

![image](/wp-content/uploads/2021/05/packer-logs-1024x203.png)

### Summary

This post was not intended to create a battle between the two ways of using packer. As you can see it all gets down to current preferences. 

AIB could maybe be the solution if you seek a lighter way to rapidly adapt and develop image creation without understanding all the packer details. This could also benefit the DevOps teams because I think simplicity is relevant to the DevOps team that needs to maintain and support further image development for the customer. To be subjective the only thing that could hold me back from using AIB, is the need for GA or Region Availability. Most WVD deployments I&#8217;m involved in requires other regions. With Managed Identity as default, this is my most preferred way to scope access control, so I like this option.  
  
The first time I used packer was related automation images to my home lab with VMware ESXi. Packer has multiple Builders for all the big players &#8211; https://www.packer.io/docs/builders. Using the Azure provider is generally different than using the VMware provider. But If you are familiar with packer, the adaption to the Builder for Azure (azure-arm) is pretty seamless. However, as listed in the table above there are also many motives why you may want to go the packer route. With the table above you can see that Packer provides the best granular and flexible control over the azure environment, you are managing. You can control &#8220;everything&#8221;. But looking back to AIB we do not need to think about specifying the packer version. I&#8217;ve encountered different errors if I&#8217;ve used an older version of packer. Some of my experience with Packer is that if you forget to validate your template or have some wrong variables this can set you back some hours of troubleshooting.  
  
That&#8217;s it for this time! Good luck with automating your AVD images in whatever way you prefer 🙂  I had a talk about this blog at AVDtechfest 2021 https://youtu.be/G2g8CVVzZP4?t=15251 and powerpoint deck https://github.com/alventech/talks/tree/main/2021/avdtechfest


### Documentation

* https://docs.microsoft.com/en-us/azure/virtual-machines/image-builder-overview>

* https://www.packer.io/docs/builders/azure/arm>

* https://github.com/azure/azvmimagebuilder/tree/main/solutions/1\_Azure\_DevOps][1]

* https://github.com/azure/azvmimagebuilder/tree/main/solutions/1_Azure_DevOps