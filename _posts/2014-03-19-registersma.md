---
layout: post
date: 2014-03-19
author: Marc van Eijk
title: Windows Azure Pack – You must first register Service Management Automation on Resource Provider VM Clouds
tags: Marc van Eijk, Service Management Automation, Service Provider Foundation, SMA, SPF, Windows Azure Pack
---
During a recent Windows Azure Pack deployment at a customer site I encountered an issue with the registration of Service Management Automation. I have done the installation and configuration numerous times without issues before. I performed the same steps at this site and the registration of SMA completed successfully. But when I wanted to link a runbook to an action in the VM Clouds resource provider I was treated with the following surprise. As you can see in the screenshot, the resource provider Automation shows the 27 sample runbooks.

<img src="/images/2014-03-19/You-must-first-register-SMA.png" width="720">

We seen some other interesting issues in this environment so I tied this inconsistency to the same list.

My fellow MVP Kristian Nese recently published a blogpost explaining [how to re-register SPF in Windows Azure Pack](http://kristiannese.blogspot.no/2014/01/troubleshooting-windows-azure-pack-re.html). You can actually use the same cmdlets to unregister SMA (or other resource providers) as well. I unregistered the SMA endpoint on the Windows Azure Pack server with the following cmdlets.

```
$Credential = Get-Credential
$Token = Get-MgmtSvcToken -Type Windows –AuthenticationSite https://yourauthenticationsite:30072 -ClientRealm http://azureservices/AdminSite -User $Credential -DisableCertificateValidation
Get-MgmtSvcResourceProvider -AdminUri “https://localhost:30004″ -Token $Token -DisableCertificateValidation -name “Automation”
Remove-MgmtSvcResourceProvider -AdminUri “https://localhost:30004″ -Token $Token -DisableCertificateValidation -Name “Automation” -InstanceId “the instance ID you got from Get-MgmtSvcResourceProvider“
```

When you verify the registration status in the admin portal after running the cmdlets you should be able to perform the registration again. I successfully registered the SMA endpoint again in the admin portal.

<img src="/images/2014-03-19/Register-SMA.png" width="500">

But the Automation tab in the VM Clouds presented the same surprise again. 
<!--more-->

After poking around with some get- cmdlets and verifying it against a working environment I found a solution. The Service Provider Foundation database is unaware of the Service Management Endpoint. I’m still looking at the root cause, but you can use the some cmdlets on the SPF server to update the SPF database with the endpoint information.

If you encounter the issue described in this blog, make sure you have the SMA endpoint registered in Windows Azure Pack and run the following cmdlets on the SPF server.

```
import-module spfadmin
Get-SCSpfStamp | fl
$stamp = get-SCSpfStamp –name “Name of the Stamp you got from the Get-SCSpfStamp”
New-SCSpfServer –name “IaasAutomation” –ServerType None –Stamps $stamp
$Server = Get-SCSpfServer –name “IaasAutomation”
New-SCSpfSetting –Name EndpointURL –SettingType EndpointconnectionString –Value “https://YourSmaEndpoint:9090/” –Server $Server
```

After a refresh the admin portal should now reflect the changes we made.

<img src="/images/2014-03-19/After-SPF-cmdlets.png" width="720">

Yesterday I got a call from [Darryl van der Peijl](http://www.darrylvanderpeijl.nl/) who was deploying a new lab environment and he encountered the exact same issue.If you also see this issue please add a comment to this blog or ping me on twitter [@_marcvaneijk](http://twitter.com/_marcvaneijk)
