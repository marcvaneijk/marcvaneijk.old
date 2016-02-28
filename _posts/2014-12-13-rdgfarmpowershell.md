---
layout: post
date: 2014-12-13
author: Marc van Eijk
title: Windows Azure Pack Remote Console – Create the RD Gateway Farm with PowerShell
tags: Marc van Eijk, Powershell, Remote Console, Windows Azure Pack, WMI
---
The community is equivalent to sharing knowledge and helping each other. One of those super motivated community members is Carsten Rachfal. I finally met him at the MVP summit. Somewhere during that week we had to walk from one building to another. I noticed has was dragging along a mobile office. Carsten explained that it contained his complete datacenter. Or to be more precise, a laptop with some crazy specs that contained the complete Cloud OS. He did a lot of work creating a complete automated installation of the the Cloud OS components with HA and perform functional configuration to end up with a environment that was demo or (if it wasn’t for the hardware) even production ready. No single click needed after the deployment process. There was one piece missing in his complete puzzle.

01 Puzzle

He had asked me a couple of times if I had a solution to complete his masterwork. But that is another thing about the community. Time. Somehow you never have enough of it. This week another reminder popped up in a DL and I forced it to the top of my priority list. His question was


I want automate the configuration of a high available RD Gateway for Windows Azure Pack Remote Console. How can I set the RD Gateway server farm members with PowerShell?

Carsten is a smart man. He has been struggling with this issue for a couple of months and it was going to complete his masterwork. He had looked at all the possible angles already.



Stubborn as I am I started looking at the PowerShell cmdlets available in the RemoteDesktop PowerShell module. I needed an environment to test with. The VM Role in our Windows Azure Pack environment came to the rescue and let me build a domain controller and two member servers in just a couple of minutes. The two member servers are called MVE-RDG1 and MVE-RDG2.

PowerShell cmdlet

In Windows Server 2012 the configuration of Remote Desktop Services changed. The installation and administration is more scenario based. The required components of the scenario will be implemented as part of a RD deployment. For our purpose (Windows Azure Pack Remote Console) we do not want a complete RD deployment, we just want two RD Gateway servers and configure them in a RD Gateway server farm for high availability. You can find the complete procedure to setup Remote Console for Windows Azure Pack here.

Enabling high availability for Remote Console is very easy to configure in the GUI. After you completed the steps described in the procedure to configure Remote Console on two servers, open the properties of the server in the Remote Desktop Gateway Manager and select the Server Farm tab. Add both servers and you are good to go.

We should be able to do this with PowerShell too, right? Looking at the commands in the RemoteDesktop PowerShell module the command named Set-RDDeploymentGatewayConfiguration. That should be it. Maybe check out the output of a Get-RDDeploymentGatewayConfiguration in a running RD Gateway server farm.

02 Get-RDGatewayDeployment


A Remote Desktop Services deployment does not exist on MVE-RDG1.inovativ.local. This operation can be performed after creating a deployment.

That was expected. Remember Carsten already looked at this numerous times. I don’t have a RD Deployment and I don’t want one either. I just want two RD Gateway servers in a farm.

When I right clicked the server in the Remote Desktop Gateway Manager I also noticed a option to Export or import policy and configuration settings. The export will create an XML file that also contains an entry called <LoadBalancingServers> containing the two servers that I added in the Server farm tab of the RD Gateway Gateway Manager.

03 XML

Mmm… maybe I’m able to automate generating that XML file with the correct server names and import it with PowerShell during a new deployment. I looked at the commands the RemoteDesktop module again. But as you know by know, Carsten in a smart man. So I wasn’t able to find a cmdlet that was able import the XML file.

Sharing knowledge

So what is the next step. Probably what everybody does. Crack open your favorite search engine and try to find some answers. This is another huge thing about the community. We all hit technical issues and challenges in our work. Somehow we manage to solve those issues. After solving an issue you gained some knowledge that is only living in your head. As long as the knowledge stays there it is completely useless. Knowledge is only powerful when shared. A lot of people share that knowledge everyday by blogging or answering questions on forums. If nobody did that, a search in your favorite engine would come up empty each time. Sharing knowledge forms the solid basis of the community. Everybody that shares his or her knowledge is automatically part of that.

I searched for a way to import the XML file with PowerShell and found this forum question. The answer was close, but I did not get the feeling this solution was very reliable. There was a pointer in the answer that got me on another track.

WMI

The answer described a by a fellow MVP contained some WMI and pointed me to an MSDN article.  The MSDN article gave some information about exporting the configuration and settings for the RD Gateway. The linked article for import even detailed different import types, where the meaning of import type with the value 4 was Import a list of all load-balancing servers. Now that is what I wanted to do. I only needed to convert the information in this MSDN article to a PowerShell script.

I suddenly remembered some cmdlets from the Remote Console article that changed a thumbprint for Remote Desktop in WMI with PowerShell.

$Server = “rdgw.contoso.com”
 $Thumbprint = “95442A6B58EB5E443313C1B4AFD2665991D354CA”
 $TSData = Get-WmiObject -computername $Server -NameSpace “rootTSGatewayFedAuth2” -Class “FedAuthSettings”
 $TSData.TrustedIssuerCertificates = $Thumbprint
 $TSData.Put()

I changed the namespace to rootCIMV2TerminalServices and the class to Win32_TSGatewayServer as specified in the MSDN import article and executed the Get-WmiObject

Get-WmiObject -Namespace rootCIMV2TerminalServices -class Win32_TSGatewayServer

04 Win32_TSGatewayServer

The output did not contain the information I was looking for. Maybe there is a different class that does contains the information about the servers in the RD Gateway server farm. Browsing through the left menu navigation on the MSDN article I found all Remote Desktop Gateway classes. The Win32_TSGatewayLoadBalancer class caught my attention. The article description was very promising.


Describes a set of Remote Desktop Gateway (RD Gateway) load-balancing servers. These are used to load balance RD Gateway connections across multiple servers.

Running the Get-WmiObject cmdlet with this class in an existing environment resulted in the output I was looking for.

Get-WmiObject -Namespace rootCIMV2TerminalServices -Class Win32_TSGatewayLoadBalancer

05 Win32_TSGatewayLoadBalancer

We are getting close. The cmdlets from the Remote Console article updated a thumbprint by performing a put(). So let’s update that script with the information we just figured out.

$RDGServers = “MVE-RDG1;MVE-RDG2”
 $RDGWLB = Get-WmiObject -Namespace rootCIMV2TerminalServices -Class Win32_TSGatewayLoadBalancer
 $RDGWLB.Servers = $RDGServers
 $RDGWLB.Put()

Running that script returned an error.

06 Put Error

Provider is not capable of the attempted operation. I was expecting something like that. The MSDN article specifies that the type server is read only and can be updated with SetServers, AddServers, DeleteServers, and DeleteAllServers methods. Another search in my favorite search engine quickly explained how to perform these methods with PowerShell. To get an list of the available methods for this class run the following cmdlet

Get-WmiObject -Namespace rootCIMV2TerminalServices -Class Win32_TSGatewayLoadBalancer | Get-Member -MemberType Method | FL

07 Methods

We have no servers in the server farm yet so the SetServer method is our weapon of choice. Before performing the script I reverted the checkpoint of the initial VM state (without any RD Gateway server farm configuration).

$RDGServers = “MVE-RDG1;MVE-RDG2”
 $RDGWLB = Get-WmiObject -Namespace rootCIMV2TerminalServices -Class Win32_TSGatewayLoadBalancer
 $RDGWLB.SetServers($RDGServers)

08 SetServers

Requesting the information again, the output reflected the new information

Get-WmiObject -Namespace rootCIMV2TerminalServices -Class Win32_TSGatewayLoadBalancer

09 SetServers Result PoSh

The Server Farm tab in the Remote Desktop Gateway Manager also shows the two server entries.

10 SetServers Result GUI

There you go Carsten, the last piece of your masterwork.
