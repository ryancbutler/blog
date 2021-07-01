---
layout: single
title: Scripts Now Available on PSGallery
joomla_id: 29
joomla_url: scripts-now-available-on-psgallery
date: 2017-08-29 16:08:17.000000000 +00:00
author: Ryan Butler
permalink: /citrix/29-scripts-now-available-on-psgallery
categories: 
  - Citrix
  - PowerShell
---

![PowerShell Gallery](/assets/images/content/psgallery/PowerShell_Gallery.png)

[Github](https://github.com/ryancbutler) is an awesome resource to share and collaborate code but sometimes not the easiest if GIT isn't installed on a Windows Server.  Lately, I have been adding some of my more popular scripts to [PSGALLERY](https://www.powershellgallery.com/profiles/ryancbutler/) which allows quick downloads and updates right from PowerShell.  In order to use PSgallery you will need to have PowerShell 5.0 installed or above and the first time you run the commands you will get prompted to install and configure the provider and modules.

![nuget](/assets/images/content/psgallery/nuget.png)

To install the script simply run the `install-script` command and agree to the prompts.  For example to install the [XDReplicate](https://github.com/ryancbutler/XDReplicate/blob/master/XDReplicate.ps1) script run

```powershell
install-script -name xdreplicate -Scope currentuser
```

NOTE: The **currentuser** scope installs under the current profile.  Otherwise the command will need to be run as administrator
{: .notice--info}

To see what scripts are installed run
```powershell
get-installedscript
```

![getinstalled](/assets/images/content/psgallery/getinstalled.png)

Now the cool thing is that you can simply update the script by running

```powershell
update-script xdreplicate
```

To check what versions exist on the repository run

```powershell
find-script xdreplicate
```

Or use tags

```powershell
find-script -tag xendesktop
```

## What scripts are available?

The following scripts have been added to PSGallery

<table border="1">

<tbody>

<tr>

<td>PSGallery Name</td>

<td>Github Repo</td>

<td>Description</td>

</tr>

<tr>

<td> get-nslicexp</td>

<td>https://github.com/ryancbutler/Citrix</td>

<td> Grabs Netscaler license expiration information via REST.</td>

</tr>

<tr>

<td> set-nsssl</td>

<td>https://github.com/ryancbutler/Citrix</td>

<td> A PowerShell script that enables TLS 1.2, disables SSLv2 and SSLv...</td>

</tr>

<tr>

<td> PVSReplicate</td>

<td>https://github.com/ryancbutler/XDReplicate</td>

<td> Checks for vDisks and versioning and will export XML if required...</td>

</tr>

<tr>

<td> XDReplicate</td>

<td>https://github.com/ryancbutler/XDReplicate</td>

<td> Exports XenDesktop site information such as administrators, deliv...</td>

</tr>

<tr>

<td> get-ICAfile_v3</td>

<td>https://github.com/ryancbutler/StorefrontICACreator</td>

<td> A Powershell v3 Script that utilizes invoke-webrequest to create,...</td>

</tr>

<tr>

<td> get-ICAfile_v3_auth</td>

<td>https://github.com/ryancbutler/StorefrontICACreator</td>

<td> A Powershell v3 Script that utilizes invoke-webrequest to create...</td>

</tr>

</tbody>

</table>