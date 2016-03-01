---
layout: post
date: 2014-01-05
author: Marc van Eijk
title: Windows Azure Pack Remote Console with the RD gateway in a DMZ
tags: Console Connect, Marc van Eijk, Remote Console, WAP, WAP Wiki, Windows Azure Pack
---
For the preview bits of Windows Azure Pack I did a [blog post](/2013/09/10/wapconsole) on connecting to a virtual machine through a console connection. The Windows Azure Pack tenant site actually allows two ways to connect to a virtual machine. The dashboard tab of a virtual machine in the tenant site present a button connect on the bottom bar. If the remote console functionality is enabled in the plan for the tenant , this button will display two options.

01 Remote Console

When you select the first option, called Desktop, you are prompted with a screen to select a virtual network interface and available IP addresses that are available to the VM. When you select the virtual network interface and IP address that you want to connect on, the IP address is injected in an RDP file that is presented to the user. The RDP file will initiate a regular RDP connection to the IP address that was injected in the RDP file. Pretty straightforward.

The second option, called Console, enables tenants to connect to their virtual machines, even if this virtual machine does not have an IP address configured, is in a blue screen or whatever reason that the default RDP connection in to the virtual machine is unavailable.

Selection this option will prompt you with a security warning advising you not to save the RDP file to a shared location. The warning is there for a reason. This RDP file, that is generated at runtime, contains the required parameters to set up a RDP session to an RD Gateway. The RD Gateway will translate it to a vmconnect session connecting to the Hyper-V host that is running the VM.

Eh.. the underlying host. But is that secure? A tenant connecting to a host? From the internet?

## Security

Yeah, it is secure! Microsoft has taken several steps to enforce security. The RD Gateway is adjusted so it can only be used for Remote Console. The settings in each RDP file that is generated is signed with a hash based on a certificate that is configured on the VMM server. The time that the RDP file is valid to initiate the session is limited and configurable by the admin. When a tenant select the Console button, access to the virtual machine for this tenant is verified and the RDP file will only be created if the tenant has the required permissions. The RDP file contains credentials that only grant access to this specific VM on the host, not to any other VMs on the host.

Configuring this functionality in the preview bits required quite some customization. With the GA of Windows Azure Pack the process of configuring Remote Console has been changed. You can find the steps to configure Remote Console in System Center 2012 R2 on Technet.

The connection between the Remote Desktop Gateway server and the Hyper-V host can be configured based on FQDN or based on IP. This setting is used when the RDP file is generated. Based on the settings the FQDN or the IP address of the Hyper-V host is added to the RDP file so the RD Gateway knows to what Hyper-V host the vmconnect session must be initiated.

## Lab environment

For a lab environment is makes sense to implement an RD Gateway as member server of the same domain as the SPF, VMM and the Hyper-V hosts.

02 Diagram LAB

In this design the RD Gateway can resolve the Hyper-V hosts on FQDN so you can use set the VMConnectHostIdentificationMode parameter to FQDN.

## Production environment

Any serious Service Provider will require all publicly accessible services to be placed in a DMZ. Besides placing all Windows Azure Pack components in a separate site and preferably in a separate domain, the RD gateway also needs to be placed in the DMZ. The RD Gateway is not able to resolve the Hyper-V hosts based on FQDN without a DNS conditional forwarder from the DMZ domain to the management domain. This also requires allowing DNS traffic from the DMZ domain to the management domain.

Remote console can also be configured to connect to the Hyper-V host on IP address. For the DMZ design, this is actually a great solution. It does not require a DNS conditional forwarder or DNS traffic allowed through the firewall. The only port that is required is TCP 2179 (for vmconnect) from the RD Gateway in the DMZ domain to the Hyper-V hosts in the management domain. In the preview of Windows Azure Pack this configuration worked excellent.

03 Diagram Service Provider

When Windows Server 2012 R2, System Center 2012 R2 and Windows Azure Pack entered GA, we reinstalled the complete environment using the RTM bits. The configuration of Remote Console is improved.  System Center VMM 2012 R2 is now in charge of the configuration on the Hyper-V hosts, which allows the design to scale for larger deployments. We configured Remote Console to connect to the Hyper-V hosts on IP address as we did in the preview bits. Performing a console connect to a virtual machine faied with the error An authentication error occurred (0x607).

607

After a lot of troubleshooting I decided to install an additional RD Gateway and make it member of the management domain. After changing the incoming NAT rule to the new RD Gateway the error remained. Finally I created a new cert, deleted all the previous certs from the stores on the involved servers, imported the cert in the DB again and refreshed the host in VMM. I could connect successfully once. I noticed that got an additional security warning with the name of the host the VM was running on. Disconnecting and connection again showed the same error again.

I got in touch with the product team and together with Ashley Travers (SDET) we looked at the environment. The issue was new for him as well. I was still questioning the certificate warning I got from the Hyper-V host the only time it worked. I proposed to change the server authentication setting setting in the auto generated RDP file from warn me to connect and don’t warn me. And that allowed us to successfully perform a Remote Console session based on IP address.

The Hyper-V host will present a self-signed certificate that contains the FQDN of the Hyper-V host. When you configure Remote Console with the VMConnectHostIdentificationMode parameter to FQDN the certificate provided by the Hyper-V host will match the initial request containing the FQDN. The tenant is prompted with the self-signed cert of the Hyper-V host and after accepting the security warning the console session is created. Configuring Remote Console with the VMConnectHostIdentificationMode parameter to IP the certificate provided by the Hyper-V host will not match the initial request containing the IP address. The error An authentication error occurred (0x607) is presented to the tenant and the console session fails.

Ashley pointed out that the line authentication level:i:0 in the RDP is used for this setting. You can look at the content of the auto generated RDP file by opening it with notepad. The entry authentication level:i:0 was present in the auto generated RDP file in the preview bits. In RTM the auto generated RDP file was slimmed down for performance but unfortunately resulted in this scenario.

## Custom Hyper-V Certificate

Ashley suggested to create a new cert for the Hyper-V host. Hyper-V requires the common name of the certificate set to the FQDN of the Hyper-V host. So we needed the certificate to contain an additional subject with the IP address of the host. The certificate requires some key usage properties to be compatible with Hyper-V. These properties can not be created with makecert so our dear friend PowerShell came to the rescue.

```
$name = new-object -com "X509Enrollment.CX500DistinguishedName.1"
$name.Encode("CN=<FQDN>, CN=<IP Address>", 0)
$key = new-object -com "X509Enrollment.CX509PrivateKey.1"
$key.ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
$key.KeySpec = 1
$key.Length = 2048
$key.SecurityDescriptor = "D:PAI(A;;0xd01f01ff;;;SY)(A;;0xd01f01ff;;;BA)(A;;0x80120089;;;NS)"
$key.MachineContext = 1
$key.Create()
$serverauthoid = new-object -com "X509Enrollment.CObjectId.1"
$serverauthoid.InitializeFromValue("1.3.6.1.5.5.7.3.1")
$ekuoids = new-object -com "X509Enrollment.CObjectIds.1"
$ekuoids.add($serverauthoid)
$ekuext = new-object -com "X509Enrollment.CX509ExtensionEnhancedKeyUsage.1"
$ekuext.InitializeEncode($ekuoids)
$kuext = new-object -com "X509Enrollment.CX509ExtensionKeyUsage.1"
$kuext.InitializeEncode(0x30)
$cert = new-object -com "X509Enrollment.CX509CertificateRequestCertificate.1"
$cert.InitializeFromPrivateKey(2, $key, "")
$cert.Subject = $name
$cert.Issuer = $cert.Subject
$cert.NotBefore = (get-date).AddDays(-5)
$cert.NotAfter = $cert.NotBefore.AddDays(900)
$cert.X509Extensions.Add($ekuext)
$cert.X509Extensions.Add($kuext)
$cert.Encode()
$enrollment = new-object -com "X509Enrollment.CX509Enrollment.1"
$enrollment.InitializeFromRequest($cert)
$certdata = $enrollment.CreateRequest(0)
$enrollment.InstallResponse(2, $certdata, 0, "")
```

This PowerShell script will create certificate in the local machine personal store.  The new certificate must be trusted by Hyper-V so it must exported to .cer and imported in the local machines trusted Root CA store. Next we will need to get the certificates thumbprint

```
$thumbprints = @(dir cert:localmachineMy | Where-Object { $_.subject -eq "CN=<FQDN of the Hyper-V host>, CN=<IP Address of the Hyper-V host>" } ).thumbprint
$thumbprints
```

Now use the certificate thumbprint in the following PowerShell script.

```
$thumbprint = "<Certificate_Thumbprint>"
$certs = dir cert: -recurse | ? { $_.Thumbprint -eq $thumbprint }
$cert = @($certs)[0]
$location = $cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName
$folderlocation = gc env:ALLUSERSPROFILE
$folderlocation = $folderlocation + "MicrosoftCryptoRSAMachineKeys"
$filelocation = $folderlocation + $location
$filelocation
icacls $filelocation /grant "*S-1-5-83-0:(R)"
$cert.thumbprint
reg add "HKLMSoftwareMicrosoftWindows NTCurrentVersionVirtualization" /v "AuthCertificateHash" /f /t REG_BINARY /d $thumbprint
reg add "HKLMSoftwareMicrosoftWindows NTCurrentVersionVirtualization" /v "DisableSelfSignedCertificateGeneration" /f /t REG_QWORD /d 1
 ```
 
This PowerShell script will configure Hyper-V to use this certificate. It also disables Hyper-V from generating its own self-signed certificate for identification. The Hyper-V service needs to be restarted. Individual Virtual Machines also require a restart, because any running VMs will maintain the existing cert even through a service restart. I have tested to move the virtual machines to another host will live migration. This will also update the certificate and prevents the virtual machines from requiring a reboot.

A Remote Console session to a virtual machine, running on the Hyper-V host, is presented with the new certificate. The new certificate contains the IP address as an additional entry in the subject and matches the IP address that was used to initiate the session from the RD Gateway. The tenant is prompted with the security warning asking if he trusts the certificate. The Remote Console session is set up successfully.

While this is a workaround for a small lab environment, it is not a solid approach for a couple of reasons. For a larger amount of Hyper-V hosts the process would quickly become difficult or even unmanageable. Every Hyper-V hosts requires its own cert with the IP address of that particular host added to the subject. If for some reason the Hyper-V host gets a new IP address assigned, a new certificate would also be required. Besides the management aspect, a tenant is asked if he trusts the Hyper-V host that is running his virtual machine when he initiated a Remote Console session. Well, off course he trusts the Hyper-V host. Otherwise he would not run his virtual machine on top of it in the first place.

## URL Rewrite

In my previous blog on Windows Azure Pack Console Connect, URL Rewrite was used to alter the RDP file that was created. Richard Rundle (Program Manager at Microsoft and owner of the Remote Console feature in Windows Azure Pack) pointed out URL Rewrite as a possible approach to add the line authentication level:i:0 to the auto generated RDP file. Now, why didn’t I think of that.

As usual what seemed a simple thing turned out to be a little more complicated.

Unfortunately it is only possible to specify a single line as the new value in URL rewrite. For example, rewrite negotiate security layer:i:1 with negotiate security layer:i:1 authentication level:i:0 ends up with both values on the same line. Richard dived in and did a hex dump of an RDP file to see what it used as a newline. He found out that it was composed of CR (0x0D) and LF (0x0A). After much trial and error with the URL Rewriter UI he decided to edit the web.config file directly and that worked. I have tested in our lab and can confirm this solution.

## Steps to configure the web.config file for Remote Console VMConnectHostIdentificationMode set to IP

Install URL Rewrite on the System Center Service Provider Foundation 2012 R2 server. URL rewrite can be installed using the same Web Platform Installer that is used for installing Windows Azure Pack.

After installer URL rewrite on the System Center Service Provider Foundation 2012 R2 server open IIS manager, select the SPF website and open the URL rewrite feature. Select Add rule to start the Add rule wizard. Select to create a blank Outbound rule, give it a temporary name, type some text in the pattern field (we will replace these settings in a second) and select apply. Open the web.config file located in the SPF folder with notepad. If you did a default installation the web.config file is located in C\:inetpubSPF. Replace the temporary outbound rule you just created with the following:

```
<outboundRules>
 <clear />
 <rule name="Remote Console on IP Address" preCondition="VMConsole RDP File" enabled="true">
  <match filterByTags="None" pattern="negotiate security layer:i:1" />
  <conditions logicalGrouping="MatchAll" trackAllCaptures="true" />
  <action type="Rewrite" value="negotiate security layer:i:1 &#xD;&#xA;authentication level:i:0" />
 </rule>
 <preConditions>
  <preCondition name="VMConsole RDP File">
   <add input="{REQUEST_URI}" pattern="^.*(/VMConnection)$" />
   <add input="{RESPONSE_CONTENT_TYPE}" pattern="^application/x-rdp$" />
  </preCondition>
 </preConditions>
</outboundRules>
```

Save the web.config file. An RDP file that is auto generated when Remote Console is after the change performed contains the authentication level:i:0 line.

05 RDP file

Remote Console with the VMConnectHostIdentificationMode set to IP now allows the tenant to connect successfully, while still complying with a Service Provider security rules. I’d like to thank Ashley and Richard for their support.

## More Information

[Remote Console in System Center 2012 R2](http://technet.microsoft.com/en-us/library/dn469415.aspx)
