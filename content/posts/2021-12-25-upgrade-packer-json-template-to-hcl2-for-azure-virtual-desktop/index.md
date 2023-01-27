---
weigth: 1
title: Upgrade Packer JSON template to HCL2 for Azure Virtual Desktop
author: Stein-Erik Alvestad
date: 2021-12-25T19:24:52+00:00
url: /upgrade-packer-json-template-to-hcl2-for-azure-virtual-desktop/
categories:
  - packer

lightgallery: true

images: []
resources:
- name: "featured-image"
  src: "DALLEcloud.png"

toc:
  auto: false

---



### Introduction

From my last blog post link /azure-image-builder-or-packer-for-wvd/ , I used a JSON packer template, and I promised a new blog post regarding how to upgrade to HCL2. 

HashiCorp recommends using the HCL2 as from version 1.7.0, so it&#8217;s about time, I finally do the switch! 

The documentation Packer is really good, so I will basically just follow along with their documentation to see If I struck any challenges. <a href="https://learn.hashicorp.com/tutorials/packer/hcl2-upgrade" target="_blank" rel="noreferrer noopener">https://learn.hashicorp.com/tutorials/packer/hcl2-upgrade</a>

During this post, we will bring the JSON template and upgrade it. I will also try to improve the template since HCL gives us better options to document annotations. 

### Getting started

![image](/wp-content/uploads/2021/12/image-4.png)

I&#8217;ve got an existing JSON file from GitHub below

{{< gist alventech 2b9a596a27bebd35e38a5edc865ad1b6 >}}

from my terminal I use the command packer hcl2\_upgrade -with-annotations .\Packer\_w10\_20h2\_SIG.json<figure class="wp-block-image size-large">

![image](/wp-content/uploads/2021/12/image-1-1024x42.png)

Successfully created .\Packer\_w10\_20h2_SIG.json.pkr.hcl. Exit 0

![image](/wp-content/uploads/2021/12/image-1024x28.png)


Time to review the HCL2 Configuration after the upgrade. I opend up the new *..pkr.hcl file in vs code to check the annotations, parameters, and variables. Working with the new template is way better and I didn&#8217;t have to change much other than the variables and Update the annotations to suit my needs. 

### Build_the_packer_HCL2_template_in_Azure_DevOps

In Github, I&#8217;ve added the raw configuration from the HCL2 upgrade, but I&#8217;ve changed the variables to my needs. 

{{< gist alventech b20fd1f1fba78dc6f89658e90b2688b6 >}}

I&#8217;m using an Azure CLI task to test the new configuration template. Go back to my old blog /azure-image-builder-or-packer-for-wvd/ if you need more details on how to configure the Azure DevOps Pipeline. 

![image](/wp-content/uploads/2021/05/packer_build_task.png)

In the Inline Script, I will add the build shell script. I’m adding the variables with export and generating timestamp for one of the variables (image_version) to set the semantic version that’s required by SIG / Azure Compute Gallery. You can also use a variable file if you prefer setting your variables that way. 

![image](/wp-content/uploads/2021/12/image-2-1024x996.png
)

Voila! When running the build we can see that it&#8217;s started the azure-arm packer Build stage.

![image](/wp-content/uploads/2021/12/image-3-1024x534.png)

### Summary

Upgrading the template was straightforward and I will for sure prefer working with the HCL configuration. If you have already been working with terraform the HCL configuration constructs more logic for you. That&#8217;s if for this short blog post. 

Documentation

* https://learn.hashicorp.com/tutorials/packer/hcl2-upgrade>

* https://www.packer.io/docs/templates/hcl_templates/variables#environment-variables>