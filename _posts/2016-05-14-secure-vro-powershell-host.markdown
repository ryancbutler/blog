---
layout: single
toc: true
toc_sticky: true
title: Secure vRO PowerShell Host
joomla_id: 19
joomla_url: secure-vro-powershell-host
date: 2016-05-14 16:58:46.000000000 +00:00
author: Ryan Butler
permalink: /vmware/19-secure-vro-powershell-host
categories:
  - VMware
tags:
  - VRO
  - SSL
  - WINRM
  - PowerShell
---
Certificates can be a pain to configure in Windows and especially when trying to use with WINRM\Powershell.  Follow this post to secure Powershell with HTTPS and add a secure "PowerShell Host" to VMware Orchestrator.

![secureps](/assets/images/content/secureps/secureps.png)

## What's Needed

For this setup you will need a Microsoft Active Directory CA setup and configured.

## CA Template

For this setup you could use an existing certificate template such as "Web Server" but for this guide we are going to create a template that is active directory integrated making things a little easier.

### Create Template

Start by logging into your CA and opening up certifcate templates.  You will need to open up MMC add the snapin "Certificate Templates".  Find the template "Web Server" and duplicate.

![temp1](/assets/images/content/secureps/temp1.png)

Let's rename the template for this purpose and extend the validity peiord if needed.

![temp2](/assets/images/content/secureps/temp2.png)

I selected to have the ability to export the private key.  Not really needed but a nice to have.

![temp3](/assets/images/content/secureps/temp3.png)

Change the mininum key size to be at least 2048

![temp4](/assets/images/content/secureps/temp4.png)

Add the required group or users that will be requesting this certificate.  I just used 'Domain Admins' for this example.

![temp5png](/assets/images/content/secureps/temp5png.png)

Set to use AD for the request and format it comes in.  This allows no additonal information required for the request.

![temp6png](/assets/images/content/secureps/temp6png.png)

Hit OK to save template

### Publish Template

Open up "Certificate Authority" from the start menu and select "Certificate Templates"

![pubtemp1](/assets/images/content/secureps/pubtemp1.png)

And select the template you created in the previous step

![pubtemp2](/assets/images/content/secureps/pubtemp2.png)

Now the certificate template is ready to go

## Powershell Host Configuration

### Certificate Request

Let's open up the certificates snapin on our Powershell Host. Run MMC and open Certificates

![cert1](/assets/images/content/secureps/cert1.png)

![cert2](/assets/images/content/secureps/cert2.png)

![cert3](/assets/images/content/secureps/cert3.png)

Now from the local computer personal store request a new certificate

![cert4](/assets/images/content/secureps/cert4.png)

Select Active directory and from the template selection check the template created earlier.

![cert5](/assets/images/content/secureps/cert5.png)

Hit enroll to request and download the new certificate

Document the certficate thumbprint for future steps.  Copy into notepad and remove spaces with find and replace.

![cert6](/assets/images/content/secureps/cert6.png)

## Bind Certificate to WINRM

We first need to enable WINRM and open firewall rules.  Open up a command prompt under administrator and run.

```bash
winrm quickconfig -transport:https
```

If there is only one certificate in the personal certificate store chances are everything is ready to go.  Run the following command and verify the the thumbprint matches.

```
winrm get winrm/config/listener?Address=*+Transport=HTTPS
```

![certverify1](/assets/images/content/secureps/certverify1.png)

If it doesn't match or if you have multiple certificates you will need to run the following commands.

Delete the current HTTPS listener

```
winrm delete winrm/config/Listener?Address=*+Transport=HTTPS
```

And now re-create with correct certificate we created.  Make sure to use the thumbprint without spaces and verify it gets copied correctly.  For some reason I have had issues where a rouge '?' gets added to the thumbprint string. (**Note the hostname is in common name format and not FQDN)**

```
winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname="iaas01";CertificateThumbprint="‎7c0cdc758bb93d70858e6352225a668eaee66da1"}
```

Now verify the listener is using the correct certificate.

```
winrm get winrm/config/listener?Address=*+Transport=HTTPS
```

If you prefer to allow remote access only from the VMware Orchestrator server run the following command

```
winrm s winrm/config/client @{TrustedHosts="vro.domain.com"}
```

If you prefer to open to all remote connections

```
winrm s winrm/config/client @{TrustedHosts="*"}
```

We are now ready to add to vRO

## Add PowerShell Host

From the vro client start the workflow "Add a PowerShell Host" and enter the information for the PS host.  Make sure to use port 5986.

![vro1](/assets/images/content/secureps/vro1.png)

Select HTTPS as the transport type, accept all certificates and make sure kerberos is selected for authentication.

![vro2](/assets/images/content/secureps/vro2.png)

When entering the credentials for the PowerShell user make sure to type in as UPN format ([username@domain.com](mailto:username@domain.com))

![vro3](/assets/images/content/secureps/vro3.png)

PowerShell host should now be successfully added and ready for some action!

If preferred, you can run the "Validate a PowerShell host" to verify

![Validate1](/assets/images/content/secureps/Validate1.png)
