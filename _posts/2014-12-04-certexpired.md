---
layout: post
date: 2010-12-04
author: Marc van Eijk
title: Windows Azure Pack authentication signing certificate expired 
tags:
---
The Cloud OS was implemented in our lab environment directly after the release of the 2012 R2 bits. That was a little over a year ago. The Windows Azure Pack installer creates multiple self-signed certificates that are used for different websites. In a simple Windows Azure Pack express installation you will get fourteen self-signed certificate. Looking at these certificates you will notice two different types. Most certificates are web server certificates assigned to a Windows Azure Pack website in IIS. There are also two signing certificates. The signing certificates are used by the Windows Azure Pack authentication sites.

<img src="/images/2014-12-04/01-Signing-Certificates.png" width="700">

I’d like to point out that one of the post deployment tasks for every environment should be to replace the default self-signed certificates with trusted certificates. This is possible for all default certificates but not for the two signing certificates used for the authentication sites.

All self-signed certificates created by the Windows Azure Pack installer have an expiration date of one year after the deployment. If you are still using self-signed certificates and they have expired after a year, you can just delete the expired certificates from the personal computer store with a certificates snap-in in an MMC and rerun the Windows Azure Pack configuration wizard after that. My fellow MVP Stanislav Zhelyazkov  has already blogged about this previously here.

Unfortunately is the self-signed authentication signing certificate recreated with the information stored in the Windows Azure Pack database, including the original expiration date. Recreating the authentication signing certificate by deleting it from the personal computer store and recreating it by running the Windows Azure Pack configuration wizard results in the same issue. An expired self-signed authentication signing certificate.

<img src="/images/2014-12-04/02-Expired-Signing-Cert.png" width="700">

After making some changes in the database I was able to recreate the certificate with a new expiration date. But as you might now, hacking the database is not supported.

Working with some smart folks from the WAP PG, we were able to convert my non supported database hacking and slashing into a supported procedure by using the following PowerShell script.

```
$Passphrase = 'Ic@nN3verR3memberMyP@ssw0rd!'
# 1. Acquire the current signing certificate thumbprint
$setting = Get-MgmtSvcSetting -Namespace AuthSite -Name Authentication.SigningCertificateThumbprint
$oldThumbprint = $setting.Value
# 2. Remove the old certificate from the global config store
# (You may need to provide additional connection string information)
Set-MgmtSvcDatabaseSetting -Namespace AuthSite -Name Authentication.SigningCertificate -Value $null -Passphrase $Passphrase -Force
# 3. Re-initialize the authentication service to generate a new signing certificate and reconfigure
# (You may need to provide additional connection string information)
Initialize-MgmtSvcFeature -Name AuthSite -Passphrase $Passphrase -Verbose
# If you are using the membership auth site directly (without ADFS trust relationship), proceed with following steps.
# If you are using ADFS with the Azure Pack, skip step 4 and go to your ADFS settings and have it "refresh" the federation metadata from the service.
# 4. Update the relying party settings for the TenantSite
$relyingPartySettings = @{
    Target = 'Tenant'
    MetadataEndpoint = "https://${env:COMPUTERNAME}:30071/FederationMetadata/2007-06/FederationMetadata.xml"
}
# (You may need to provide additional connection string information)
Set-MgmtSvcRelyingPartySettings @relyingPartySettings -DisableCertificateValidation
# 5. (optional) Remove the old signing certificate no-longer in use
Get-Item Cert:\LocalMachine\My\$oldThumbprint | Remove-Item -Force -Verbose
# 6. Reset services to update any (old) cached configuration
iisreset
# 7. Attempt to login to TenantSite to verify service is working
Start-Process "https://${env:COMPUTERNAME}:30081"
```
 
