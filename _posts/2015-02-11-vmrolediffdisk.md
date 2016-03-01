---
layout: post
date: 2015-02-11
author: Marc van Eijk
title: Windows Azure Pack VM Role – Choose between differencing disks or dedicated disks
tags: Dedicated Disks, Differencing Disks, IaaS, Marc van Eijk, VM Role, Windows Azure Pack
---
Windows Azure Pack was released in October 2013 and allows you to provide cloud services that are running in your own datacenter. Since its release we have deployed a lot Cloud OS environments. Most if not all deployments contained or were centered around Infrastructure As A Service.

To enable infrastructure as a service in your datacenter you need a couple of components.

- Windows Azure Pack
- System Center Service Provider Foundation
- System Center Virtual Machine Manager
- Hypervisor (almost all Windows Azure Pack IAAS functionality requires Hyper-V)

As a tenant in the Windows Azure Pack portal you can interact with virtual machines and virtual networks.

For deploying virtual machines you can choose between two methods.

- Stand alone virtual machine
- VM Role

<img src="/images/2015-02-11/GalleryItem.png" width="700">

### Stand alone virtual machine

The stand alone virtual machine is a one to one mapping to a VM Template in Virtual Machine Manager. The properties of the stand alone virtual machine live in VMM and can only be changed there. The stand alone virtual machine can be used to deploy a virtual machine with an operating system. The deployment wizard in Windows Azure Pack is easy and straight forward but cannot be customized. You are bound by the options in the existing wizard and the capabilities of the VM Template in Virtual Machine manager.

### VM Role

The other method Windows Azure Pack provides to deploy virtual machines is the VM Role. The VM Role uses the service template engine in Virtual Machine Manager and combines that with a customizable deployment wizard in Windows Azure Pack. On top of the stand alone virtual machine method the VM Role provides the following capabilities

- Application deployment in the virtual machine as an integral part of the deployment process
- Customizable deployment wizard
- Better interaction capabilities with Service Management Automation
- Deploy and manage a single tier of one or multiple instances.
- Servicing of the application through tenant configuration
- Versioning of the VM Role with application updating capabilities

Stand alone virtual machine or the VM Role? Now this looks like an easy choice. And every customers reaction to this comparison is similar. The VM Role it is.

But…, there is one important thing to point out. The VM Role uses differencing disks.

## Differencing disks

A differencing disk is a virtual hard disk you use to isolate changes to a virtual hard disk or the guest operating system by storing them in a separate file. A differencing disk is associated with another virtual hard disk that you select when you create the differencing disk. This means that the disk to which you want to associate the differencing disk must exist first. This virtual hard disk is called the “parent” disk and the differencing disk is the “child” disk. The parent disk can be any type of virtual hard disk (fixed or dynamically expanding). The differencing disk stores all changes that would otherwise be made to the parent disk if the differencing disk was not being used. The differencing disk provides an ongoing way to save changes without altering the parent disk. Multiple child disks can use the same parent disk.

The VM Role uses differencing disks for its virtual hard disks. A VM Role consists of one Operating System disks and optionally one or more data disks. In the VM Role configuration you define information (metadata) about each disk of that VM Role. The metadata is a family name and a version number. Additional filtering for the Operating System disk can be set with tags.
<!--more-->

The VMM library must contain a sysprepped Operating System virtual hard disk, with the same family name, version number and tags. It must also be configured with an Operating System property. If the VM Role contains data disks, these data disk must exist in the VMM library, with a unique family name , a version number and the Operating System Property must be set to None. Tags are not used with Data Disks.

With the release or System Center R2 support for differencing disks is added to Virtual Machine Manager. The properties of a Hyper-V host contains a Placement Paths tab.

<img src="/images/2015-02-11/Placement-Paths.png" width="700">

When you deploy your very first VM Role with only an Operating system disk, VMM is used to locate the sysprepped virtual hard disk in the library and copy that file to the location that is configured in the Placement Paths for the host selected through the intelligent placement process in VMM. If no default parent disk path is set, the location for the parent disk is defined through VMM intelligent placement.

When the parent disk is copied from the library to the target location, an entry to that path is configured in the VMM database for that host. The deployment process creates a child disk on top of the parent disk and uses the child disk for the virtual machine instance.

### Scaling

A VM Role can be scaled. You can add or remove virtual machine instances within the single tier. The scaling process is very easy for a tenant. By moving a slider, the number of instances can be increased or decreased within the boundaries set in the VM Role configuration. When a tenant increases the number of virtual machine instances of a deployed VM Role, the differencing disk mechanism is used to boost the performance of the scaling action. VMM checks the database to verify if the parent disk already exists on target storage and if the host has access to it. If that is true and that host matches the other intelligent placement requirements a child disk is created on the same parent disk. This saves time and space by preventing another copy of the parent disk.

### Data Disk

A disk in a VM Role that contains a data disk in its configuration is subject to the same placement rules. If the very first VM Role is deployed that contains a data disk the disk is copied from the VMM library to the target location. This location, combined with the host, is added as an entry to the VMM database. A differencing disk is created on top of the parent data disk and used as the data disk for the VM Role. Scaling the VM Role will create a new child disk on top of the same parent disk that is also used for the first instance of the VM Role.

### More VM Roles

Deploying additional VM Roles will also use the differencing disk configuration. As an example, if you have ten different VM Roles, where each VM Role references the same Operating System disk and you deploy each VM Role ten times to a single host or Hyper-V cluster that has access to the same storage, you will end up with a single parent disk and one hundred child disks.

### Advantage of differencing disks

When a tenant deploys a virtual machine from Windows Azure Pack, he expect the virtual machine to be available in a relative short amount of time. Copying virtual hard disks from the VMM library to the target destination consumes bandwidth and takes time. Using differencing disk for the VM Role improves the deployment speed by removing the disk copy step from process (unless new disks are introduced in the configuration).

### Challenges with differencing disks

A differencing disk configuration will have different Storage I/O characteristics, since the Parent disk will be used by different virtual machines. This can be acceptable for Operating System disks, but when you are deploying multiple high performance workloads to differencing data disks, like SQL or Exchange, you can run into issues. Some workloads might not even support differencing disks.

Managing and troubleshooting differencing disk can be a complex task. We discussed the example of ten VM Roles with the same Operating System disk configuration, each deployed ten times to a single host. Ending up with one parent disk and one hundred data disks. But what if you have multiple VM Role configurations with different Operating System and data disks deployed to multiple clusters with different storage. You will end up with a mesh of child to parent disk dependencies that will become even more complicated if you start moving virtual machines around between storage locations with Storage Live Migration. Hyper-V replica will ignore the multiple child to parent relationship and create a single parent and child at the target site for each replicated disk. And when looking at windows updates for the Operating System disk residing in the VMM library, do not even consider offline servicing of that sysprepped disk. As you will risk ending up with two different parent disks on two storage locations that VMM will treat as equal and valid for a storage live migration. Which will result in a corrupt VM.

### Tradeoff and workarounds

Despite the many challenges with the differencing disk configuration of the VM Role, a lot of organizations still accept these tradeoffs because of the benefits the VM Role itself provides them. And inventive as we are, we figure out ways to work around these challenges, for example by converting the differencing disk or attaching disks as an additional step after the deployment.

## Microsoft listens to you

With the release of Update Rollup 5 for Virtual Machine Manager, Microsoft has made the necessary changes to allow you to choose your disk configuration. You can use still use differencing disks, but it is now also possible to use a dedicated disk configuration for each virtual machine instance. The choice is made by you and configured at the VMM cloud level. This functionality does not have any affect in VMM unless you use it in combination with the VM Role.

One of the most important reasons Microsoft has enabled this update is because you voted for [this feedback](http://feedback.azure.com/forums/255259-azure-pack/suggestions/6036538-option-to-select-between-differencing-disk-or-dedion) on the user voice. If you want to influence the priorities of the product group you can vote on existing suggestions or submit your own suggestions here.

- Windows Azure Pack: <http://feedback.azure.com/forums/255259-azure-pack>
- Virtual Machine Manager: <https://systemcentervmm.uservoice.com/forums/280803-general-vmm-feedback>

## How to enable dedicated disks

After installing Update Rollup 5 for Virtual Machine Manager you can enable dedicated disks for VM Role by setting a custom property on a cloud in VMM. After the custom property has been set each VM Role that is deployed to that cloud uses dedicated disks. Existing VM Roles that were deployed to the same cloud before the property was set will keep using the differencing disks, even when it is scaled.

A plan in Windows Azure Pack is mapped to a cloud in VMM. This allows you to choose between differencing disks or dedicated disks for VM Role at the plan level. Each new VM Role deployed in a subscription that is mapped to that plan will be configured with the disk type based on the custom property on the cloud in VMM, that the plan is mapped to.

To enable a cloud in VMM for VM Roles with dedicated disks op the properties of the Cloud in the VMs and Services tab of VMM. Select the Custom Properties tab.

<img src="/images/2015-02-11/Cloud.png" width="700">

Select Manage Custom Properties and in the windows that opens click create.

Specify the following name for the custom property

`DifferencingDiskOptimizationSupported`

Add the newly created custom property as an assigned property. Verify the object type is set to cloud.

<img src="/images/2015-02-11/Custom-Property.png" width="700">

Click OK to submit and assign the new custom property.

The final step is to specify a `false` value for this custom property.

<img src="/images/2015-02-11/Custom-Property-value.png" width="700">

New VM Role that are created in a plan that is mapped to this VMM cloud will be configured with dedicated disks.

## Fixed Disks or Dynamically Expanding Disks

A question I also get frequently is does the VM Role use fixed disks or dynamically expanding disks? Well you can actually choose. It supports both. The VM Role will copy the source file from the VMM library. So whatever you choose for the source disk (fixed or dynamically expanding) that is what the target VM Role will use. So you could even mix both types in one instance. For example dynamically expanding for the Operating System disk and fixed for the data disks that are used for the SQL databases and log files.

### On a personal note

You have this awesome solution but there is one tiny thing that is holding you back and you try to squeeze it in by doing all kind of workarounds.

<img src="/images/2015-02-11/squeeze.png" width="700">

A lot of people at Microsoft have been very helpful in getting a fitting garage for the VM Role supercar. A big THANK YOU to Eric Winner, Stephen Baron, John Ballard, Bradley Bartz, Robert Reynolds, Hector Linares and especially David Armour, Pradeep Reddy and Manish Jha for their time, listening ear and hard work.

## More Information

- [Description of the security update for Update Rollup 5 for System Center 2012 R2 Virtual Machine Manager](http://support2.microsoft.com/kb/3023195/en-us)
- [VM Role Series on Building Clouds Blog](http://blogs.technet.com/b/privatecloud/archive/tags/marc+van+eijk/default.aspx)
