---
layout: post
date: 2014-03-09
author: Marc van Eijk
title: Windows Azure Pack – VM Role custom Virtual Machine sizes
tags: CloudVMRoleSizeProfile, Marc van Eijk, Service Template, VM Role, VM template, Windows Azure Pack
---
In Windows Azure Pack you can deploy standalone virtual machines, which are directly mapped to VM Templates in Virtual Machine Manager. The VM Templates are limited to deploy an Operating System, without applications. The VM Template allows you to configure the number of CPUs and the amount of Memory assigned to it. Since the standalone Virtual Machine in Windows Azure Pack is a direct mapping to the VM Template in Virtual Machine Manager, a virtual machine that is deployed by a tenant will be configured according to the size you specified in the VM template. If you would like to give a tenant the possibility to change the number of CPUs and amount of Memory assigned to virtual machine you can define hardware profiles in Virtual Machine Manager and add them to the plan that the tenant has a subscription on.

<img src="/images/2014-03-09/00-Standalone-Virtual-Machine.png" width="720">

Virtual Machine Manager also provides Service Templates. Service Templates use VM Templates as building blocks and add a lot of functionality on top of them. Scale-up, Scale-out, application integration, relation to deployed instances and versioning, just to name a few.

With the release of Windows Azure Pack a new feature called the VM Role was introduced. The VM Role is a Windows Azure Pack gallery item, that uses the service template engine in Virtual Machine Manager. It allows you to deploy a virtual machine with automated application installation. Most of the functionality that you might be familiar with from Service Templates are also present in the VM Role. For a good comparison have a look at [this blog](http://blogs.technet.com/b/privatecloud/archive/2013/12/11/how-we-developed-vmrole-gallery-items-from-service-templates.aspx).

The VM Role consists of two parts. The Resource Definition is imported in to Windows Azure Pack and contains the fields to build up the deployment wizard and map to the Resource Extension. The Resource Extension is imported in to Virtual Machine Manager and contains the application logic. Virtual Machine Manager does not provide a Graphical User Interface for The Resource Extension. You can create new or edit existing Resource Definition and Resource Extension files with the [VM Role Authoring Tool](https://vmroleauthor.codeplex.com/).

When you deploy your first VM Role, you will notice that the available sizes for the virtual machine are populated automatically.
<!--more-->

<img src="/images/2014-03-09/Untitled.png" width="720">

The predefined list contains the following sizes.

Name | Description  | CpuCount | MemoryInMB 
--- | --- | --- | ---
ExtraSmall | Extra Small Size VM | 1 | 768 
Small | Small Size VM | 1 | 1792 
Medium | Medium Size VM | 2 | 3584 
Large | Large Size VM | 4 | 7168 
ExtraLarge | Extra Large Size VM | 8 | 14336 
A6 | A6 Size VM | 4 | 28672 
A7 | A7 Size VM | 8 | 57344 

The VM Role is also present in Windows Azure. If you have deployed a virtual machine in Windows Azure before you will immediately recognize these sizes. To minimize the differences for moving the VM Roles between Windows Azure Pack and Windows Azure, it makes sense to match the VM sizes between the two environments. But what if you have a company policy that dictated other VM sizes or you are Service Provider and would like to provide your own VM sizes. The VM Authoring Tool also has a JSON tab to show the actual code.

I played around with some adjustments in code but could not find a way. A ping to the VM Role (and also Service Template) Guru, Mr. Baron at MSFT provided the solution. He directed me to a PowerShell cmdlet.

## CloudVMRoleSizeProfile

The list of VM sizes for the VM Role are predefined in Virtual Machine Manager. You can retrieve the existing values by running the following command on the VMM server.

```
Get-CloudVMRoleSizeProfile | ft Name, Description, CpuCount, MemoryInMB –AutoSize
```

<img src="/images/2014-03-09/02-Get-CloudVMRoleSizeProfile.png" width="720">

If you want to make changes to the predefined list it might me a good idea to first export the existing values. You can export the values in CSV format by running the following cmdlet (make sure the folder exists before running the cmdlet).

```
Get-CloudVMRoleSizeProfile | Export-Csv c:\export\DefaultCloudVMRoleSizeProfiles.txt
```

<img src="/images/2014-03-09/03-Export.png" width="720">

If you want to restore the default values at some point in time (for whatever reason), you can use the values from the generated file.

<img src="/images/2014-03-09/04-Csv.png" width="720">

I will add a VM size with the following values as an example. Take note that the Name parameter can only contain alphabet letters, numbers, underscores(_), and dashes(-) and has a maximum value of 100 characters.

Name | KillerVM 
--- | ---
Description | This VM will empty your creditcard in a couple of minutes 
Number of CPUs | 32 
Amount of Memory | 131072 

Use the following cmdlet to add this VM size

```
New-CloudVMRoleSizeProfile -Name "KillerVM" -Description "This VM will empty your creditcard in a couple of minutes" -CPUCount 32 -MemoryMB 131072
```

<img src="/images/2014-03-09/05-New-CloudVMRoleSizeProfile.png" width="720">

Logon to the admin portal and deploy a new VM Role. The new VM size will show up as an option.

<img src="/images/2014-03-09/06-NewSizeinPortal.png" width="720">

To remove an existing VM size run the following command. Replace the Name value with the entry you would like to remove.

```
Get-CloudVMRoleSizeProfile -Name "KillerVM" | Remove-CloudVMRoleSizeProfile
```

<img src="/images/2014-03-09/07-Remove-CloudVMRoleSizeProfile.png" width="720">

The built-in CloudVMRoleSizeProfiles cannot be removed with the Remove-CloudVMRoleSizeProfile. It is possible to change these built-in profiles with the Set-CloudVMRoleSizeProfile. You are able to set CpuCount, MemoryInMB and Description. The name cannot be changed. To change the CpuCount, MemoryInMB and Description for the built-in ExtraLarge profile run the following cmdlet

```
Get-CloudVMRoleSizeProfile -name ExtraLarge | Set-CloudVMRoleSizeProfile -CPUCount 10 -Description "Some Other Description" -MemoryMB 28672
```

## More Information

[Windows Azure Pack (#WAPack) and Related Blogs, Videos and TechNet Articles](http://social.technet.microsoft.com/wiki/contents/articles/20689.windows-azure-pack-wapack-and-related-blogs-videos-and-technet-articles.aspx)
