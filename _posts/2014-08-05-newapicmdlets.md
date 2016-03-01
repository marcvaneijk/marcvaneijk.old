---
layout: post
date: 2014-08-05
author: Marc van Eijk
title: Windows Azure Pack Tenant Public API new cmdlets
tags: Powershell, Tenant Public API, VM Role, Windows Azure 
---
Two months ago I published a [blog](/2014/06/11/tenantpublicapi) on the Windows Azure Pack Tenant Public API. This API allows you to interact with your cloud services using PowerShell over the internet and certificate authentication. The Microsoft Azure PowerShell module provided cmdlets for Windows Azure Pack as well. As you might remember from that blog was the lack of VM Role cmdlets. There was a workaround that worked but was somewhat complex to configure and maintain.

A new version of the Microsoft Azure PowerShell module has been released. This new version also contains various new cmdlets  for Windows Azure Pack. 
- New-WAPackCloudService 
- Get-WAPackCloudService 
- Remove-WAPackCloudService 
- New-WAPackVMRole 
- Get-WAPackVMRole 
- Set-WAPackVMRole 
- Remove-WAPackVMRole 
- New-WAPackVNet 
- Remove-WAPackVNet 
- New-WAPackVMSubnet 
- Get-WAPackVMSubnet 
- Remove-WAPackVMSubnet 
- New-WAPackStaticIPAddressPool 
- Get-WAPackStaticIPAddressPool 
- Remove-WAPackStaticIPAddressPool 
- Get-WAPackLogicalNetwork

As you can see it also contains new cmdlets for interacting with cloud services and the VM Role.

You can download Microsoft Azure PowerShell module 0.8.6 through the Web Platform Installer with [this link](http://go.microsoft.com/?linkid=9811175&clcid=0x409).

The VM Role is a custom configuration that can consist of many required and optional fields. As with the GUI wizard some values must be provided for the PowerShell cmdlet. Creating a new VM Role with the New-WAPackVMRole cmdlet requires some input.

<img src="/images/2014-08-05/01wapcmdlet.png" width="700">

If we take a look at the ResourceDefinition of an existing VM Role there is still some configuration requirement, but it is a huge improvement compared the previous procedure.

<img src="/images/2014-08-05/02resdef.png" width="700">
