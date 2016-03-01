---
layout: post
date: 2014-06-11
author: Marc van Eijk
title: Windows Azure Pack Tenant Public API
tags: API, Marc van Eijk, Microsoft Azure, Powershell, PublishSettings, Tenant Public API, Tenant Site, Windows Azure Pack
---
Microsoft Azure and Windows Azure Pack are like two circles. These circles are moving towards each other and are already overlapping on certain parts. The CloudOS vision is those two circles completely merged into one.

Circles

So, when you work with Windows Azure Pack it is very interesting to keep an eye on Microsoft Azure, the public cloud solution from Microsoft. This gives a good idea of the features that are coming to Windows Azure Pack, but also gives more insight in the features that are already available in Windows Azure Pack today. In this blog we will cover a feature that is not very well known but can be very useful. The Windows Azure Pack Tenant Public API.

Most Windows Azure Pack deployments we see in production are in one way or another related to IaaS. Windows Azure Pack provides a powerful web portal that enables tenants to interact with their IaaS services. They can create, edit and delete Virtual Machines and Virtual Networks with just a few clicks.

The tenant portal experience is awesome, but there are scenarios where other methods are required. Take for example regression testing. A tenant want to schedule a deployment for a set of virtual machines with applications. When the virtual machines are deployed, an automated procedure runs tests against the applications, which logs the performed steps to a location for evaluation. After the tests are completed the virtual machines are decommissioned again. The regression tests are scheduled by the tenants and they make changes to the tests frequently.

The first thing that comes to mind with this example is a combination of the VM Role and Service Management Automation. The VM Role allows you to deploy a virtual machine with an application. Service Management Automation enables scheduling of PowerShell workflows that can deploy the VM Roles for the tenant and run the regression tests as well.

Unfortunately in this release of Windows Azure Pack you need access to the Windows Azure Pack Admin Site to edit or schedule an SMA runbook. This requires Admin interaction for each change in the runbook or each change in the schedule, which is not an option.

Microsoft Azure provides a powerful scripting environment with Azure PowerShell. It allows tenants to interact with the services in their Microsoft Azure subscription with PowerShell cmdlets. These cmdlets can be run from a remote client. The client authenticates to the services in the subscription by using certificates. As you expect from Microsoft Azure it works after some easy steps to get the certificates configured correctly.

WAPack

If you have a closer look at the cmdlets within the Azure PowerShell module you will notice that there are also cmdlets that contain WAPack in their name. This looks promising. 

## Tenant Public API

Windows Azure Pack is installed through the Web Platform Installer. Windows Azure Pack consists of many components. If you are looking for guidance on the placements of all the roles I suggest you have a look at the recording of this TechEd session I did, that explains all the components and where to place them in your environment. The Tenant Public API is provided directly to the tenant, so it should be placed on a server or tier that is accessible to the tenant. The most common design is to place this component on the same server or tier as the Windows Azure Pack Tenant Site.

The Tenant Site and the Tenant Public API will have their own URLs both hosted on SSL port 443 with the same public wildcard web server certificate. We will use the Server Name Indication feature in IIS 8.5 to provide a host header with SSL for the Tenant Site. Please read this blog for guidance on how to configure the Tenant Public API on port 443.

## Prepare the client

After succesful configuration of Windows Azure Pack, a tenant needs to prepare his client machine to interact with his services though PowerShell. The following steps must be performed to prepare the client environment for running PowerShell cmdlets against the Windows Azure Pack Tenant Public API.
Install the Windows Azure PowerShell module
Download and import the publishsettings file

### Install the Windows Azure PowerShell module

By default the Windows Server and Windows Client Ooperating system contain a predefined set of PowerShell modules. Additional modules can be installed. Microsoft Azure provides a PowerShell module that also contains cmdlets for Windows Azure Pack. You can download the latest version of the module through the Web Platform Installer. In the Web Platform Installer select Windows Azure PowerShell and select Add. Click Install.

Instead of accepting the license terms, click the link “click here to see additional software to be installed and review the associated Microsoft license terms” In the popup click the “Direct Download Link”

WEBPI

The download should now only contain the Windows Azure PowerShell module (about 8-9 MB in size).

Download

Install this module.

### Download and import the publishsettings file

The Windows Azure Pack API uses certificate authentication for communication from the client to the API. When a tenant perform the following procedure, a certificate with a public key is generated and stored in the Windows Azure Pack database for every subscription that the tenant owns. The same certificate with the private key is imported in the client environment. Based on these certificates the client environment has access to the resources within his subscriptions.

To configure the certificates a publish settings file needs to be downloaded from the tenant portal. Logon on to the Windows Azure Pack Tenant Site. In the example from my previous blog the URL is <https://manage.hyper-v.nu>.

Select My Account in the left menu and open the Management Certificates entry. Verify that no certificates are present.

NoCerts

Now enter the URL for the Tenant Site again but append `/publishsettings` to it. In this example the URL is <https://manage.hyper-v.nu/publishsettings>. A custom page will be displayed and you are prompted to download a file. Download the file. Please note that this file contains the hash to create the certificate with the private key.

DownloadPublishSettingsFile

When the file is downloaded a certificate with a private key is hashed and configured within the downloaded file. The same certificate with only the public key is imported in to the Windows Azure Pack database. This certificate is now displayed in the tenant portal. My account > Management Certificates.

Certs in Portal

Open a PowerShell console on the client where the file was downloaded and where the Windows PowerShell module was also installed and run the following command.

```
# Import the PublishSettings file (downloaded from https://manage.hyper-v.nu/publishsettings)
Import-WAPackPublishSettingsFile "C:\<path of the downloaded file>\<name of the file>.publishsettings"
```

This import process will create a certificate with the private key (containing the same thumbprint as the cert that was created in Windows Azure Pack) in the user personal certificate store. After importing the downloaded publishsettings file should be deleted (for security reasons).

MMC

You can now access the subscription in Windows Azure Pack with PowerShell based on certificate authentication. The certificate authentication is still valid after closing Internet Explorer, PowerShell or even a reboot.

## Windows Azure Pack PowerShell examples

After the client environment is prepared for Windows Azure Pack PowerShell the user can interact with his subscriptions through PowerShell cmdlets.
 
```
# List of Windows Azure Pack commands (contained in the Microsoft Azure PowerShell module)
get-command *wapack*

# Get a list of the subscriptions
Get-WAPackSubscription | ft SubscriptionName, SubscriptionId -AutoSize

# Select another subscriptionGet-WAPackSubscription -SubscriptionName "hypervnu" | Select-WAPackSubscription

# Get the available Networks
Get-WAPackVNet | ft Name, IsolationType, LogicalNetworkName, ID -AutoSize

# Get the available Single Instance VM templates
Get-WAPackVMTemplate | ft Name, OperatingSystem, CPUCount, Memory -AutoSize
```
 
Windows Azure Pack provides two types of Virtual Machines. The Stand Alone Virtual Machine and the Virtual Machine Role. The Stand Alone Virtual Machine is a direct mapping to the VM Template in System Center Virtual Machine Manager. It allows for a Operating System deployment within virtual machine. The Virtual Machine Role leverages the Service Template engine in System Center Virtual Machine Manager to deploy virtual machines with an operating system and applications.

### Stand Alone Virtual Machine

The Windows Azure PowerShell module contains a cmdlet for the Stand Alone Virtual Machine.

```
## Create a VM in the current subscription
$VMName = "MyVM"
$Template = Get-WAPackVMTemplate -Name "Windows Server 2012 R2"
$VNet = Get-WAPackVNet -Name "hypervnu_network"
$MyCred = Get-Credential

New-WAPackVM -Name $VMName -Template $Template -VNet $VNet -VMCredential $MyCred -Windows

## Start, stop and remove an existing VM
Get-WAPackVM -Name "MyVM" | Start-WAPackVM
Get-WAPackVM -Name "MyVM" | Stop-WAPackVM
Get-WAPackVM -Name "MyVM" | Remove-WAPackVM
```

### Virtual Machine Role

Update August 5, 2014: A new version of the Microsoft Azure PowerShell module now contains cmdlets for the VM Role. See more [here](/2014/08/05/newapicmdlets).

~~The Windows Azure PowerShell module does not (yet) contains a cmdlet for the Virtual Machine Role.~~ Luckily for us Charles Joy posted a great workaround which converts PowerShell to JSON with the ConvertToJson cmdlet. This results in a more complex script but does allow for Virtual Machine Role deployment.
 
```
#region GetWAPConnectionData

# Get WAP Subscription Information
$WAPSubscription = Get-WAPackSubscription -SubscriptionName "hypervnu"

# Set Subscription
$SubscriptionID = $WAPSubscription.SubscriptionId

# Get Management Certificate Info
$CertThumb = $WAPSubscription.Certificate.Thumbprint
$CertPath = "Cert:\CurrentUser\My\{0}" -f $CertThumb
$Cert = Get-Item $CertPath

# Set Tenant Portal Address
$TenantPortalAddress = $WAPSubscription.ServiceEndpoint.Host

# Set Port
$Port = $WAPSubscription.ServiceEndpoint.Port

#endregion GetWAPConnectionData
```
 
Change the variables to the correct values in your environment.

```
#region SetVariables

# Set Gallery Item Name and Version for Match and Deploy
$GalleryItemName = "SQLServer2012"
$GIVersion = "1.0.0.0"

# Set Common Gallery Item Parameters
$UserID = "marcvaneijk@hyper-v.nu"
$VMRoleNetwork = "hypervnu_network"
$CloudServiceName = "CloudService-4-{0}" -f $SubscriptionID
$VMRoleNamePattern = "MyVM##"
$VMRoleName = "SQLServer"
$VMRoleSize = "Small"
$VMRoleTZ = "W. Europe Standard Time"
$OSDisk = "Windows Server 2012 R2 Standard"
$OSDiskVersion = "1.0.0.0"
$RunAsDomainToJoin = "hypervnu.local"
$DomainNETBIOSName = "hypervnu"
$SafeModeAdminPassword = "P@$$word"

#endregion SetVariables
```
 
I added a cmdlet here to list the available VM Roles and a additional match operator to prevent issues if multiple versions from a single VM Role are available.
 
```
#region GetResDef

# Get Gallery Item Reference
$GIReferenceUri = "https://{0}:{1}/{2}/Gallery/GalleryItems/$/MicrosoftCompute.VMRoleGalleryItem?api-version=2013-03" -f $TenantPortalAddress,$Port,$SubscriptionID
$GIReferenceData = [xml](Invoke-WebRequest -Certificate $Cert -Uri $GIReferenceUri | Select-Object -ExpandProperty Content)

#### List VM Roles
$GIReferenceData.feed.entry.content.properties | ft name, label, publisher, version, description -AutoSize

#### Select VM Role
$GalleryItemREF = $GIReferenceData.feed.entry.content.properties.resourcedefinitionUrl | ? {$_ -match $GalleryItemName -and $_ -match $GIVersion}

# Get Gallery Item Resource Definition
$GIResDEFUri = "https://{0}:{1}/{2}/{3}/?api-version=2013-03" -f $TenantPortalAddress,$Port,$SubscriptionID,$GalleryItemREF
$GIResourceDEFJSON = Invoke-WebRequest -Certificate $Cert -Uri $GIResDEFUri | Select-Object -ExpandProperty Content

#Convert ResDef JSON to Dictionary
[System.Reflection.Assembly]::LoadWithPartialName("System.Web.Extensions") | Out-Null
$JSSerializer = New-Object System.Web.Script.Serialization.JavaScriptSerializer
$ResDef = $JSSerializer.DeserializeObject($GIResourceDEFJSON)

#endregion GetResDef
```
 
The parameter names in the Hashtable from the following region must meet the names specified in the VM Role Resource definition. You can verify these values in the Authoring tool.

Authoring Tool

```
#region SetResDefConfig

# Create Gallery Item Parameter Hashtable (for Common Data)
$GIParamList = @{
VMRoleVMSize = $VMRoleSize
VMRoleOSVirtualHardDiskImage = "{0}:{1}" -f $OSDisk,$OSDiskVersion
VMRoleTimeZone = $VMRoleTZ
VMRoleComputerNamePattern = $VMRoleNamePattern
VMRoleNetworkRef = $VMRoleNetwork
RunAsDomainToJoin = $RunAsDomainToJoin}

# Add Parameter to Hashtable (for VM Role specific values)
$GIParamList += @{Windows2012R2DCDomainDNSName = $UserID.Split("@")[1]} 
$GIParamList += @{Windows2012R2DCDomainNETBIOSName = ($UserID.Split("@")[1]).Split(".")[0]}
$GIParamList += @{Windows2012R2DCSafeModeAdminPassword = $Password}

# Convert Gallery Item Parameter Hashtable To JSON
$ResDefConfigJSON = ConvertTo-Json $GIParamList
#Add ResDefConfig JSON to Dictionary
$ResDefConfig = New-Object 'System.Collections.Generic.Dictionary[String,Object]'
$ResDefConfig.Add("Version",$GIVersion)
$ResDefConfig.Add("ParameterValues",$ResDefConfigJSON)

#endregion SetResDefConfig
```

```
#region GenerateGIPayloadJSON

# Set Gallery Item Payload Variables
$GISubstate = $null
$GILabel = $VMRoleName
$GIName = $VMRoleName
$GIProvisioningState = $null
$GIInstanceView = $null

# Set Gallery Item Payload Info
$GIPayload = @{
"InstanceView" = $GIInstanceView
"Substate" = $GISubstate
"Name" = $GIName
"Label" = $GILabel
"ProvisioningState" = $GIProvisioningState
"ResourceConfiguration" = $ResDefConfig
"ResourceDefinition" = $ResDef}

# Convert Gallery Item Payload Info To JSON
$GIPayloadJSON = ConvertTo-Json $GIPayload -Depth 7

#endregion GenerateGIPayloadJSON
```

```
#region GetOrSetCloudService

# Get Cloud Services
$CloudServicesUri = "https://{0}:{1}/{2}/CloudServices?api-version=2013-03" -f $TenantPortalAddress,$Port,$SubscriptionID
$CloudServicesData = [xml](Invoke-WebRequest -Uri $CloudServicesUri -Certificate $Cert | Select-Object -ExpandProperty Content)
$CloudService = $CloudServicesData.feed.entry.content.properties.Name | ? {$_ -match $CloudServiceName}
if (!$CloudService) {

# Set Cloud Service Configuration
$CloudServiceConfig = @{
"Name" = $CloudServiceName
"Label" = $CloudServiceName
}

# Convert Cloud Service Configuration To JSON
$CloudServiceConfigJSON = ConvertTo-Json $CloudServiceConfig
$CloudServicesData = [xml](Invoke-WebRequest -Uri $CloudServicesUri -Certificate $Cert -Method Post -Body $CloudServiceConfigJSON -ContentType "application/json")
$CloudService = $CloudServicesData.entry.content.properties.Name | ? {$_ -match $CloudServiceName}
}

#endregion GetOrSetCloudService
```

```
#region DeployGIVMRole

# Set Gallery Item VM Role Deploy URI
$GIDeployUri = "https://{0}:{1}/{2}/CloudServices/{3}/Resources/MicrosoftCompute/VMRoles/?api-version=2013-03" -f $TenantPortalAddress,$Port,$SubscriptionID,$CloudService

# Deploy Gallery Item VM Role
$VMRoleDeployed = Invoke-WebRequest -Uri $GIDeployUri -Certificate $Cert -Method Post -Body $GIPayloadJSON -ContentType "application/json"
$VMRoleDeployed

#endregion DeployGIVMRole
```

The PowerShell Script is divided into regions for easier reference. When the deployment is started successfully the output is similar to this.

```
StatusCode : 201
StatusDescription : Created
Content : <?xml version="1.0" encoding="utf-8"?><entry xml:base="https://spf.hyper-v.nu:8090/SC2012R2/VMM/Microsoft.Management.Odata.svc/"
xmlns="http://www.w3.org/2005/Atom" xmlns:d="http://schemas....
RawContent : HTTP/1.1 201 Created
x-ms-request-id: 555de8a8-df2b-4d60-aa09-e5dc4d190de7
x-ms-orchestrator-job-id: ce738a9f-a8c4-40f3-82e3-1f1c55e7f2a0
X-Content-Type-Options: nosniff
request-id: 4bdaf69b-70b4-...
Forms : {}
Headers : {[x-ms-request-id, 555de8a8-df2b-4d60-aa09-e5dc4d190de7], [x-ms-orchestrator-job-id, ce738a9f-a8c4-40f3-82e3-1f1c55e7f2a0], [X-Content-Type-Options, nosniff], [request-id, 4bdaf69b-70b4-0001-c607-db4bb470cf01]...}
Images : {}
InputFields : {}
Links : {}
ParsedHtml : System.__ComObject
RawContentLength : 11066
```

## Troubleshooting

Repeating the certificate creation process multiple time will result in certificate mismatches. To remove the complete configuration, perform the following steps.

- Delete the certificate(s) in the Windows Azure Pack portal
- Delete the certificate(s) in the personal entry of the user certificate store (with certificates MMC snap-in)
- Delete the WindowsAzureProfile.xml file from %appdata%\Windows Azure PowerShell
- Repeat the process described in Prepare the client.

The deployment of the VM Role fails. Validate the following steps.

- Can you request a list of the virtual machines in the subscription? If you can it is not related to the Windows Azure Pack API configuration.
- Validate the name of the VM Role specified in the $VMRoleName variable does not match the name of an existing deployed instance in the Windows Azure Pack subscription.
- Are the variables in the Common Gallery Item Parameters entry referencing the correct values in the VM Role?
- Does the Create Gallery Item Parameter Hashtable contain all the values required by the design of the Virtual Machine Role and do the names match with the names defined in the Virtual Machine Role JSON file?

The System Center Virtual Machine Manager logfile will provide additional information to solve the deployment issue.

## More Information

- [The Windows Azure Pack Wiki (#WAPack)](http://social.technet.microsoft.com/wiki/contents/articles/20689.the-windows-azure-pack-wiki-wapack.aspx)
- [Automation–The New World of Tenant Provisioning with Windows Azure Pack (Part 3): Automated Deployment of the Identity Workload as a Tenant Admin](http://blogs.technet.com/b/privatecloud/archive/2014/03/12/automation-the-new-world-of-tenant-provisioning-with-windows-azure-pack-part-3-automated-deployment-of-the-identity-workload-as-a-tenant-admin.aspx)
- [Github – Azure Powershell](https://github.com/Azure/azure-sdk-tools#windows-azure-pack)
