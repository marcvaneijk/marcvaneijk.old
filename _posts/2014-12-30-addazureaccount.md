---
layout: post
date: 2014-12-30
author: Marc van Eijk
title: Add-AzureAccount cmdlet connects with incorrect user ID
tags: Add-AzureAccount, Azure Active Directory, Marc van Eijk, Microsoft account, Microsoft Azure, Organizational account, Powershell
---
Since a couple of months I’ve been getting my hands dirty with Azure Resource Manager. PowerShell is one of the possible ways to interact with this awesome new feature that Microsoft has introduced to the Azure platform. As you probably know, the Azure PowerShell module allows you to connect and interact with your resources in Azure. You can connect to your Azure subscription from PowerShell with certificate based authentication or with Microsoft Azure Active Directory authentication.

For most situations I use Microsoft Azure Active Directory authentication. The procedure for logging on to your subscription is easy. Install the latest PowerShell module and run the Add-AzureAccount cmdlet. You will be prompted with a interactive login page.

<img src="/images/2014-12-30/01-Logon.png" width="700">

The sharp observer will notice that someone at Microsoft needs to put a penny in a jar every time the Add-Azure cmdlet is used.

You can use your Microsoft account or Organizational account to login without the need of any management certificate or publish settings file. A description of both account types can be found [here](http://msdn.microsoft.com/en-us/subscriptions/dn531048.aspx).

> Organizational account is an account created by an organization’s administrator to enable a member of the organization access to all Microsoft cloud services such as Microsoft Azure, Windows Intune or Office 365. An Organizational account can take the form of a user’s organizational email address, such as username@orgname.com, when an organization federates or synchronizes its Active Directory accounts with Azure Active Directory.
> 
> Microsoft account, created by a user for personal use, is the new name for what used to be called “Windows Live ID”. The Microsoft account is the combination of an email address and a password that a user uses to sign in to all consumer-oriented Microsoft products and cloud services such as Outlook (Hotmail), Messenger, OneDrive, MSN, Windows Phone or Xbox LIVE. If a user uses an email address and password to sign in to these or other services, then the user already has a Microsoft account. But the user can also sign up for a new one at any time.

I have two different accounts. One account is an Organizational account and the other is a Microsoft account.
<!--more-->

<img src="/images/2014-12-30/02-msn.png" width="700"> The Microsoft account is used for creating a subscription in Azure.

<img src="/images/2014-12-30/03-marcvaneijk.com_.png" width="700"> The Organizational account is used for creating an office 365 subscription. I also created a Azure subscription with this account.

Besides these two personal accounts I also have a business email address.

<img src="/images/2014-12-30/04-inovativ.png" width="700"> This business email address is not enabled for any Microsoft services whatsoever.

For provisioning a resource group through PowerShell I executed the Add-AzureAccount cmdlet and authenticated with my Microsoft account.

<img src="/images/2014-12-30/05-Logon-Microsoft-Account.png" width="700">

The cmdlet returns an ID, Type, Subscriptions and Tenants. To interact with Azure Resource Manager you need to switch to the Azure Resource Manager module by running the following cmdlet.
 
```
Switch-AzureMode -Name AzureResourceManager
```

To retrieve a list of the currently available  Azure Resource Manager templates.
 
```
Get-AzureResourceGroupGalleryTemplate
```

But instead of getting a list of templates, I got an error.

Your Azure credentials have not been set up or have expired, please run Add-AzureAccount to set up your Azure credentials.

<img src="/images/2014-12-30/06-Error.png" width="700">

To retrieve the information of the account used in for session run

```
Get-AzureAccount
```

The output of the command confused me. The ID was set to my business email address.

<img src="/images/2014-12-30/07-Get-AzureAccount.png" width="700">

As specified before, this account is not enabled for any Microsoft services whatsoever. So how did I end up with with this user ID for my PowerShell session?

## Troubleshooting steps

I started by removing all associated environments, subscriptions and accounts in the PowerShell session by running

```
Clear-AzureProfile
```

Performed the same Add-AzureAccount procedure and validation with my Microsoft account again. It resulted in the same user ID mismatch. Is this a problem with the Azure PowerShell module, my machine or something in my subscription? Quickly deployed a new VM from our Windows Azure Pack lab environment and installed the latest PowerShell (0.8.11) module on it. Performed the Add-AzureAccount on the new VM with the same User ID mismatch as result. So the issue was not caused by machine.

A quick search on the net did not return any similar issues and I know I’m not the only on using the Add-AzureAccount cmdlet. The logical next step was to look at my Azure subscription. This subscription is used for demo purposes, therefore I decided to cancel my subscription and create a new one with the same user account. Performed the Add-AzureAccount, again with the same User ID mismatch as result. So the issue was not caused by my subscription either.

In the cool new Azure preview portal something caught my attention. The dropdown on the user account showed three directories.

<img src="/images/2014-12-30/08-Directories.png" width="700">

The first directory was part of my subscription, but the second directory was part of the subscription created by the Office 365 account. And the third directory really raised my eyebrows.

**microsoft.onmicrosoft.com**

This was getting more confusing by the minute. After getting a good night sleep it hit me the next morning. Office 365 uses Azure AD for its user accounts. In SharePoint online (part of Office 365) you can invite external users. The external user requires a Microsoft account or an Organizational account to accept the invite. The account the invite is sent to is mapped to the account used to accept the invite.

The user administration page in my Office 365 Organizational account revealed my Microsoft account. To test the theory I deleted my Microsoft account from the Office 365 user administration page, logged on with my Microsoft account to http://portal.azure.com and checked the directories again. The directory from my Office 365 subscription was gone.

<img src="/images/2014-12-30/09-Two-Directories.png" width="700">

Running the Add-AzureAccount resluted in one less tenant, but the account mismatch was still there.

I still didn’t understand why my business email address showed up in the Add-AzureAccount. After some more troubleshooting I selected the microsoft.onmicrosoft.com directory in the new azure portal and to my surprise my business email address was displayed.

<img src="/images/2014-12-30/04-inovativ.png" width="700">

Since this email address was not enabled for Microsoft services whatsoever, it was obvious that the issue correlated to an external user in Office 365. I work with Microsoft on a regular basis. For a project I was invited to a SharePoint site by Microsoft. The invite was sent to my business email address and I accepted that invite with my Microsoft account. Which caused my business email business address to be mapped to my Microsoft account in the microsoft.onmicrosoft.com directory.

## Solution

Unlike my own Office 365 subscription, I do not have access to the user administration pages of the microsoft.onmicrosoft.com directory (would be nice dough). To prevent the user mismatch with Add-AzureAccount just create a new user in your Azure AD, add that user as a co-administrator to the subscription and use that user for authenticating in the Add-Azure cmdlet. That was the easy way.

I took the hard way. Reached out to one of the PMs owning the Azure PowerShell module. I explained my findings to him by sharing my screen in a Lync session. Two days later the issue was [fixed](https://github.com/Azure/azure-powershell/compare/334b96ace8ee2db5b204d7b79ebfdde3a865f934...v0.8.12-December2014). Kudos!! The fix is added in the latest version of the Azure PowerShell module (0.8.12) that you can download with the Web Platform Installer.

Happy new year!!
