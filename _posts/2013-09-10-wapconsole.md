---
layout: post
date: 2013-09-10
author: Marc van Eijk
title: Windows Azure Pack Console Connect
tags: Certificate, claims based authentication, Console Connect, Marc van Eijk, Remote Desktop Gateway, URL rewrite, VMConnect, WAP, Windows Azure Pack
---
Almost a year ago I wrote a blog on the Beta of [Windows Azure for Windows Server](/2012/11/09/wa4ws).

It was a very promising solution, but also had some shortcomings from a Service Provider perspective. I summed up a couple of requirements or nice to haves at the end of that blog

- TCP Endpoints
- Site to site VPN
- VM Networks
- Remote in to VM RDP Gateway with single sign on
- Change owner of existing virtual machines
- Change SPF Registration URL and credentials

A lot of great work has been done since the RTM version of Windows Azure Services for Windows Server. Not only have they implemented all these requirements for Service Providers, but they even pushed the envelope by implementing a lot of other great features. Because all these new functionalities deserve an easier and more clear name, the product is renamed to Windows Azure Pack.

<img src="/images/2013-09-10/One-Platform.png" width=720">

Microsoft has released the Preview bits for Windows Azure Pack for download through the Web Platform Installer that can be downloaded [here](http://www.microsoft.com/web/downloads/platform.aspx).

In the previous version (Windows Azure Services for Windows Server) the only way to connect to a virtual machine was by using the Remote Desktop Protocol and connect directly to the IP address of the virtual machine. This required that the virtual machine was running, RDP in the guest OS was enabled, firewall exclusions for RDP were in place and the virtual machine had a public IP address. This public IP address would then be injected in to an RDP file when a tenant used the connect button from the Service Management Portal.

If one of these requirements for connecting to the virtual machine was not met, you were unable to connect in to the virtual machine. You probably can think of some situations where this would be the case.

One of the great new features in Windows Azure Pack is console connect. This feature allows you to connect to a virtual machine through the underlying host. This enables you to connect to the virtual machine as you are used to by using VMConnect from the Hyper-V Manager console, but then over SSL without direct access to the host.

So no matter if your virtual machines has a private IP address, no IP address, a crashed operating system or no operating system at all, you are able to connect to it and take the required steps to resolve these issues. To connect to a virtual machine with Console Connect in the Preview version of Windows Azure Pack it is required that the client that initiates the Console Connect session is running Windows 8.1 Preview.

There are some limitations to Console Connect when you compare it the a default Remote Desktop connection. It is not possible to use clipboard, sound, printer redirection and drive mapping.

There are a lot of folks out there trying the Preview bits and having a hard time to get this configured. After a chat with the Program Manager for this feature we agreed that the steps to get Console Connect configured can be described in this blog post. Please take note that these step will change in the RTM for Windows Azure Pack, where the host management for Console Connect is moved to System Center Virtual Machine Manager 2012 R2.

## Security

Console Connect leverages the Remote Desktop Gateway feature in Windows Server 2012 R2. In essence you are connecting to a host that will in turn present the screen/keyboard/mouse functionality (optionally through a Remote Desktop Gateway). This requires some proper security measures to enforce that a tenant can only access his own virtual machines and not the virtual machines from another tenant or even worse connect to the host from the management domain.

To enforce these security measures, support for claims based authentication in Windows Server 2012 R2 Hyper-V is leveraged. Windows Azure Pack and System Center Service Provider Foundation 2012 R2 authenticate and authorize access to virtual machines and provide a token that the Hyper-V host uses to grant access to a single virtual machine. Certificates are used to create a trust relationship between the Hyper-V hosts and System Center Service Provider Foundation 2012 R2. The certificate allows claims tokens issued by System Center Service Provider Foundation 2012 R2 to be accepted by the Hyper-V hosts.

In addition to the security measures with claims based authentication the configuration of the Remote Desktop Gateway will limit its functionality to be used only for console access to virtual machines and unable to be used for other purposes.

## Design

For this blog I’m using a lab environment. In this lab environment System Center Virtual Machine Manager 2012 R2 and the Windows Server 2012 R2 Hyper-V clusters are located in a management domain. Windows Azure Pack is installed in a DMZ in a separate domain. The Remote Desktop Gateway is deployed in the same domain as Windows Azure Pack. The two environments are separated by a firewall. If you have a smaller lab environment with a single domain and all the Azure Pack components installed on a single host (express installation) the steps for configuring Console Connect will be the same except for the location of some objects. I will assume that you have Windows Azure Pack already up and running.

<img src="/images/2013-09-10/Design.png" width=720">

The steps for configuring Console Connect for Windows Azure Pack preview, can be divided into the following parts.

- Certificate
- Service Provider Foundation
- Hyper-V hosts
- Remote Desktop Gateway (only required for access from the internet)
- Windows Azure Pack

## Certificate

A certificate with a private key on System Center Service Provider Foundation 2012 R2 is used for claims based authentication. The public keys of this certificate must be installed on the Hyper-V hosts and the Remote Desktop Gateway to accept tokens signed by System Center Service Provider Foundation 2012 R2.

Valid certificates for issuing remote console tokens have the following required attributes:

- The certificate has not expired
- The Key Usage field contains Digital Signature
- The Enhanced Key Usage contains Client Authentication (1.3.6.1.5.5.7.3.2)
- The root CA certificates of the CAs that issued this certificate must be installed in the Trusted Root Certification Authorities store

For the lab environment I’m using a self-signed certificate, but you can also a the Enterprise Certificate Authority if you have one configured in your domain or request a certificate from a public Certificate Authority.

Note: If you use a self-signed certificate it is necessary to place the public key of the certificate into the Trusted Root Certification Authorities store on the Remote Desktop Gateway and the Hyper-V hosts.

For creating the self-signed certificate you can use makecert.exe that is part of the [Windows Software Development Kit (SDK) for Windows 8.1 Preview](http://msdn.microsoft.com/en-us/library/windows/desktop/bg162891.aspx). If you have an older version of makecert lying around make sure that is capable of creating certificates with an sha256 algorithm. You can verify this my running 

```
makecert.exe -! -a <algorithm> (The signature's digest algorithm) <md5|sha1|sha256|sha384|sha512> (Default to 'sha1')
```

The result should display sha256 as possible options. I have placed a copy of the makecert file from the Windows Software Development Kit (SDK) for Windows 8.1 Preview [here](https://skydrive.live.com/redir?resid=A49424DE2FED3A2!1349&authkey=!AHCBtSsbeR2P5vA)

Create a self-signed certificate by running:

```
makecert -n "CN=Remote Console Connect" -r -pe -a sha256 -e <mm/dd/yyyy> -len 2048 -sky signature -eku 1.3.6.1.5.5.7.3.2 –sr "LocalMachine" -ss My -sy 24  "<CertificateName>.cer"
```

-sky signature | use for signing 
--- | --- | ---
-r | create self-signed 
-n "CN=Remote Console Connect" | subject name (Remote Console Connect) 
-pe | private key is exportable 
-a sha256 | cert algorithm 
-len 2048 | key length 
–e <mm/dd/yyyy> | expiry date 
-eku 1.3.6.1.5.5.7.3.2 | enhanced key usage (client auth) 
-ss My | place the private key in certificate store (My) 
-sy 24 | Cryptographic provider type (supports SHA256) 
"<CertificateName>.cer" | name for the public key 

The command generates a self-signed certificate in the certificates user store. To export the certificate open a Microsoft Management Console (mmc.exe) add the certificates snap-in and connect to the user account. Expand Certificates > Personal > Certificates, right-click the certificate we just created, open the All Tasks menu and choose export.

In the Export Private Key screen select No, do not export the private key. In the Export File Format screen choose to export the certificate to a Base-64 encoded X.509 (.CER) file. Specify the location to export the certificate and complete the wizard.

Start the Certificate Export Wizard again. In the Export Private Key screen select Yes, export the private key. This forces the wizard to export the certificate as Personal Information Exchange – PKCS # 12 (.PFX) file. Leave the default checkmark on the Export File Format screen and click next. Enable the Password checkmark on the Security screen and specify a password for the .PFX file. Specify the location to export the certificate and complete the wizard. This exported certificate (.PFX file) with the private key will be installed on the server running System Center Service Provider Foundation 2012 R2. The .CER file will be installed on the server running System Center Service Provider Foundation 2012 R2, the Remote Desktop Gateway and the Hyper-V hosts.

## Service Provider Foundation

Logon to the server running System Center Service Provider Foundation 2012 R2 and copy the .PFX and .CER files we just created. Import the .PFX file by double-clicking on it. Select Local Machine as the Store Location, confirm the file path, enter the password you used when you exported the certificate, optionally select to mark the key as exportable, select to place the certificate in the following store and browse to the Personal store.

The import process of the .PFX file can also be done by running the following PowerShell script.

```
Import-PfxCertificate -CertStoreLocation Cert:LocalMachineMy -Filepath "<certificate path>.pfx" -Password <Secure String>
```

When the .PFX file is imported, start the import of the .CER file by double-clicking on it. Select install certificate on the General tab. Select Local Machine as the Store Location, select to place the certificate in the following store and browse to the Trusted Root Certification Authorities store.

The import process of the .CER file can also be done by running the following PowerShell script.

```
Import-Certificate -CertStoreLocation cert:LocalMachineRoot -Filepath "<certificate path>.cer"
```

System Center Service Provider Foundation 2012 R2 needs to be configured to use the certificate to create claims tokens by using the Set-SCSPFVmConnectGlobalSettings cmdlet.

We need to specify three values for the cmdlet.

1.The lifetime value of a token. The command allows you to Issue tokens that have a lifetime between 1 and 60 minutes.
2.You can specify how hosts are identified. Host can be identified by; 
 - FQDN – fully qualified domain name
 - Host – hostname
 - IPv4 – IPv4 address
 - IPv6 – IPv6 address
3.The thumbprint of the certificate we created earlier.

We can find the certificate thumbprint by running the following cmdlet.

```
$thumbprints = @(dir cert:localmachineMy | Where-Object { $_.subject -eq "CN=Remote Console Connect" } ).thumbprint
$thumbprints
```

In my lab environment the Remote Desktop Gateway is located in a separate domain from the Hyper-V hosts and there is no name resolution from the DMZ domain to the management domain. In this blog I will configure Console Connect to use the IP address of the host to connect to.

Run the following cmdlets to configure System Center Service Provider Foundation 2012 R2 to use the certificate to create claims tokens.

```
import-module spfadmin
Set-SCSPFVmConnectGlobalSettings -AccessTokenLifetimeInMinutes 5 -CertificateThumbprint 9D726C513A9A5D358CE13F5E1682052C61A4C7DF -HostIdentityType IPv4`
```

You can verify your settings by running the cmdlet.

```
Set-SCSPFVmConnectGlobalSettings
```

In the Windows Azure Pack preview bits there is a known issue with the RDP file that is created for a tenant when using Console Connect. To workaround this issue you need to install URL Rewrite on the System Center Service Provider Foundation 2012 R2 server. URL rewrite be installed using the same Web Platform Installer that is used for installing Windows Azure Pack.

After installer URL rewrite on the System Center Service Provider Foundation 2012 R2 server open IIS manager, select the SPF website and open the URL rewrite feature. Select Add rule to start the Add rule wizard. Select to create a blank Outbound rule, give it a temporary name, type some text in the pattern field (we will replace these settings in a second) and select apply. Open the web.config file located in the SPF folder with notepad. If you did a default installation the web.config file is located in C:inetpubSPF. Replace the temporary outbound rule you just created with the following:

```
<outboundRules>
 <clear />
 <rule name="Fix gatewayaccesstoken" preCondition="VMConsole RDP File">
  <match filterByTags="None" pattern="gatewayeaccesstoken" />
  <conditions logicalGrouping="MatchAll" trackAllCaptures="true" />
  <action type="Rewrite" value="gatewayaccesstoken" />
 </rule>
 <rule name="Fix gatewaycredentialssource" preCondition="VMConsole RDP File">
  <match filterByTags="None" pattern="gatewaycredentialssource:i:4" />
  <conditions logicalGrouping="MatchAll" trackAllCaptures="true" />
  <action type="Rewrite" value="gatewaycredentialssource:i:5" />
 </rule>
 <rule name="Fix gatewayprofileusagemethod" preCondition="VMConsole RDP File">
  <match filterByTags="None" pattern="gatewayprofileusagemethod:i:0" />
  <conditions logicalGrouping="MatchAll" trackAllCaptures="true" />
  <action type="Rewrite" value="gatewayprofileusagemethod:i:1" />
 </rule>
 <preConditions>
  <preCondition name="VMConsole RDP File">
   <add input="{REQUEST_URI}" pattern="^.*(/VMConnection)$" />
   <add input="{RESPONSE_CONTENT_TYPE}" pattern="^application/x-rdp$" />
  </preCondition>
 </preConditions>
</outboundRules>
```

Save the web.config file and perform an IISReset.

In a default configuration the SPF service account does not have access to the private key of the Console Connect certificate that is installed on the SPF server.

After configuring all parts and the tenant uses Console Connect, the ManagementODataService log on the SPF server displays the following error. Operation manager plugin method ‘GetReadStream’ for resource name ‘VMM.VirtualMachine’ failed with error messsage ‘Keyset does not exist’.

Add the SPF domain service account to the local administrator group on the SPF server and a reboot the SPF Server.

## Hyper-V hosts

On each Hyper-V host, that will run virtual machines accessible by Windows Azure Pack, the following configuration steps must be done. As noted before these steps are specific for Windows Azure Preview. In Windows Azure Pack RTM this configuration is moved to System Center Virtual Machine Manager 2012 R2.

First we need to import the public key of the certificate. Copy the .CER file we created earlier and start the import of the .CER file by double-clicking on it. Select install certificate on the General tab. Select Local Machine as the Store Location, select to place the certificate in the following store and browse to the Trusted Root Certification Authorities store. Performs these import steps on each Hyper-V host.

The import process of the .CER file can also be done by running the following PowerShell script.

```
Import-Certificate -CertStoreLocation cert:LocalMachineRoot -Filepath "<certificate path>.cer"
```

Restart the Hyper-V Virtual Machine Management Service after importing the certificate on each Hyper-V host. The certificate should be imported in the certificate store on the Hyper-V servers before continuing.

When authenticating tokens Hyper-V will only accept tokens that are signed using specific certificates and hashing algorithms. The TrustedIssuerCertificateHashes property is an array of certificate thumbprints. To configure the TrustedIssuerCertificateHashes property use the following cmdlet.

```
$Server = "RES-HVT01.res.local"
$Thumbprint = "9D726C513A9A5D358CE13F5E1682052C61A4C7DF"
$TSData = Get-WmiObject -computername $Server -NameSpace "rootvirtualizationv2" -Class "Msvm_TerminalServiceSettingData"
$TSClass = Get-WmiObject -computername $Server -NameSpace "rootvirtualizationv2" -Class "Msvm_TerminalService"
$TSData.TrustedIssuerCertificateHashes = $Thumbprint
$JobResult = $TSClass.ModifyServiceSettings($($TSData.GetText('CimDTD20'), $null))
```

Replace the $Server variable with the FQDN of the host where you are running the script on.

Replace the $Thumbprint variable the thumbprint of the certificate we create earlier. The cmdlet to retrieve the thumbprint is the same that we used before

```
$thumbprints = @(dir cert:localmachineMy | Where-Object { $_.subject -eq "CN=Remote Console Connect" } ).thumbprint
$thumbprints
```

## Remote Desktop Gateway

In this example the Remote Desktop Gateway is located in the DMZ domain. It is also possible to locate the Remote Desktop Gateway in the same domain as all other components. When you have installed the Operating System (in this example Windows Server 2012 R2), install the Remote Desktop Gateway using Add Roles and Features in Server Manager.

Install the Microsoft System Center Virtual Machine Manager Console Connect Gateway component on the Remote Desktop Gateway server by running RDGatewayFedAuth.msi from CDLayout.EVALamd64SetupmsiRDGatewayFedAuth located on the System Center Virtual Machine Manager 2012 R2 Evaluation media.

After the installation completes, execute the following command.

```
regsvr32 "C:Program Files\Microsoft System Center 2012 R2\Virtual Machine Manager\Console Connect Gateway\bin\fedauthplugin.dll"
```

When authenticating tokens the Remote Desktop Gateway will only accept tokens that are signed using specific certificates and hashing algorithms. The TrustedIssuerCertificateHashes property is an array of certificate thumbprints. This configuration is performed by setting the TrustedIssuerCertificate properties on the WMI FedAuthSettings class by using the following cmdlet.

```
$Server = "rdgw.contoso.com"
$Thumbprint = "9D726C513A9A5D358CE13F5E1682052C61A4C7DF"
$TSData = Get-WmiObject -computername $Server -NameSpace "rootTSGatewayFedAuth2" -Class "FedAuthSettings"
$TSData.TrustedIssuerCertificates = $Thumbprint
$TSData.Put() 
```

Replace the $Server variable with the FQDN of the host where you are running the script on.

Replace the $Thumbprint variable the thumbprint of the certificate we create earlier. The cmdlet to retrieve the thumbprint is the same that we used before

```
$thumbprints = @(dir cert:localmachineMy | Where-Object { $_.subject -eq "CN=Remote Console Connect" } ).thumbprint
$thumbprints
```

The Authentication plugin should be set with the following PowerShell cmdlet.

```
$g = Get-WmiObject -Namespace rootCIMV2TerminalServices -Class Win32_TSGatewayServerSettings
$g.SetAuthenticationPlugin("FedAuthAuthenticationPlugin")
$g.SetAuthorizationPlugin("FedAuthAuthorizationPlugin")
$g.RecycleRpcApplicationPools()
```

And finally create this registry key on the Remote Desktop Gateway server

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Terminal Server Gateway\preRDP\DisableRegKey
```

The Remote Desktop Gateway should be configured with a web server certificate provided by a public Certificate Authority. To configure this certificate you can use the default configuration steps for assigning a certificate in the Remote Desktop Gateway management console.

If you have located the Remote Desktop Gateway server in a separate domain like I have, then you also need to punch a hole in the firewall. To enable connectivity a tenant needs to access the Remote Desktop Gateway from the internet over port 443 and the Remote Desktop Gateway will need access to the Hyper-V hosts on TCP port 2179.

## Windows Azure Pack

Now with all components in place we need to configure the plan in Windows Azure Pack preview to provide the Console Connect functionality to Tenant. Take note that a change to a plan in Windows Azure Pack preview will change the status of the Plan to syncing indefinitely. Create a new plan and assign the virtual machine clouds service. Open the plan and select to configure the virtual machine clouds plan service. On the bottom of the page is an option to enable connect to the console of virtual machines.

<img src="/images/2013-09-10/Plan.png" width=720">

When you enable this option the tenant will get the option to connect to the console of a virtual machine from the Service Management Portal.

<img src="/images/2013-09-10/ConsoleConnect.png" width=720">

The tenant is presented with an RDP file in the bottom of the page.

<img src="/images/2013-09-10/RDP-File.png" width=720">

After opening the file the tenant is prompted with an unknown publisher warning from the virtual machine itself. When the tenant click connect the remote desktop connection is made and the tenant will see the console of the virtual machine. A good shortcut to know is ALT + CTRL + END to perform an CRTL + ALT + Delete in the Console Connect window.

<img src="/images/2013-09-10/Console-Connect-screen.png" width=720">

Hopefully the top bar on the Console Connect window will be updated for Windows Azure Pack RTM, because it now displays the IP address (since we choose to connect on IP) of the underlying host. I don’t think most Service Provider will be to happy providing that information to the tenant.

## More Information

- [For more information on Windows Azure Pack check out these videos from TechEd 2013](http://channel9.msdn.com/Events/TechEd/Europe/2013/MDC-B364#fbid=htLx5u4w1Fe)
- [Enabling On-Premises IaaS Solutions with the Windows Azure Pack](http://channel9.msdn.com/Events/TechEd/Europe/2013/MDC-B364#fbid=htLx5u4w1Fe)
- [Administering and Automating On-Premises IaaS Scenarios Using the Windows Azure Pack](http://channel9.msdn.com/Events/TechEd/Europe/2013/MDC-B362#fbid=htLx5u4w1Fe)
- [Building Cloud Services with Windows Server 2012 R2, Microsoft System Center 2012 R2 and the Windows Azure Pack](http://channel9.msdn.com/Events/TechEd/Europe/2013/MDC-B215#fbid=htLx5u4w1Fe)
- [Enabling Multi-Tenant IaaS Clouds in Microsoft System Center and Windows ServerEnabling Multi-Tenant IaaS Clouds in Microsoft System Center and Windows Server](http://channel9.msdn.com/Events/TechEd/Europe/2013/MDC-B318#fbid=htLx5u4w1Fe)
