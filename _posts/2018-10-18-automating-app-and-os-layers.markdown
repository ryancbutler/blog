---
layout: single
title: Automating App and OS Layers
joomla_id: 34
joomla_url: automating-app-and-os-layers
date: 2018-10-18 03:30:55.000000000 +00:00
author: Ryan Butler
permalink: /citrix/app-layering/34-automating-app-and-os-layers
toc: true
toc_sticky: true
categories:
  - Citrix
  - App Layering
tags:
  - SDK
  - PowerShell
  - App Layering
  - Unidesk
---
Back in May I attended my local CUGC where Ron Oglesby presented a master class on Citrix App Layering (ELM) and after the presentation, it was asked if there was any sort an SDK. Ron explained that there is no public API or SDK (yet), but some of the newer ELM components were built with this in mind. This statement resonated with me and when I got home that afternoon, I fired up Fiddler to see how the API worked. After a rather lengthy process of reverse engineering the API with Fiddler, Postman and documenting the processes I was able to write my SDK. When I completed the SDK I wasn’t really sure how it could be utilized in an environment or if there would be a need. Figured it would be more useful around documenting an ELM environment vs actually automating the creation or updating layers. I was under the impression that if an organization could automate layer creation, chances are they could automate the entire image making ELM redundant. However, after thinking about it for some time I now see ELM in combination with automation potentially being the stopgap between manual installs\updates and a fully automated build. By automating certain aspects of ELM it could allow organizations to tiptoe into automation and create a comfort level and free up time otherwise spent on updating common layers. This allows administrators to continue manually installing the difficult applications or even controlled by other teams but allowing automation to keep the common apps (e.g. Firefox, Chrome, Notepad Plus Plus…) and OS layers fully updated on regular intervals.

![layer](/assets/images/content/sdk/layer.png)

The intent of this series of posts is how to approach automating some common apps and keeping the operating system layers fully updated.

1.  The first section will detail the steps needed for the script host and the base operating system layer allowing for automation.
2.  The second section will cover automatically building and updating app layers utilizing Choclatey packages.
3.  The third will cover continually keeping an operating system layer updated with Windows updates

## Disclaimer

Just some notes before getting started.

*   The SDK in these posts isn’t supported by Citrix so please TEST fully and make sure you understand what's taking place.
*   These guides are by no means the only way to approach automating ELM but a proof a concept to show it can be done and others can hopefully use as a base.
*   The process isn’t necessarily the most secure but again this is to show the possibilities so please run with care.
*   I really hope Citrix sees the value in having a supported SDK and the possibilities it can create to further the adoption of ELM.
{: .notice--warning}

## Requirements

To get started you’ll need the following configured and ready to go.

1.  vCenter Environment and access credentials (If you run AHV, XenServer or Hyper-v you’ll have to rework the scripts.)
2.  Citrix App Layering Appliance fully configured
3.  A new Windows 10 Operating System Layer or version
4.  This process should work with other versions of Windows just not tested.
5.  A “script host” to run the scripts from. (This can be either a server or pc OS).
6.  Download scripts found [here](https://github.com/ryancbutler/UnideskSDK/tree/master/Examples)

## Script Host and OS Layer Preparation

Starting off, we will go over the configuration of the OS layer and script host to allow the scripts to successfully run.

![diagram](/assets/images/content/sdk/diagram.png)

### Script Host

The script host communicates with the ELM update VMs via WinRM. In order for this communication to take place, the script host needs to be allowed to access the update VM. The * allows access to any IP. This post does not go into restricting access further but there are plenty of guides outlining this.

Run as admin

```powershell
set-wsmaninstance -ResourceURI winrm/config/client -ValueSet @{TrustedHosts="*"}
```

Let’s install both my ELM SDK and VMware PowerCLI

```powershell
Find-Module -Name VMware.PowerCLI  
Install-Module -Name VMware.PowerCLI  
Find-Module -Name ctxal-sdk  
Install-Module -Name ctxal-sdk
```

### Operating System Layer

An operating system layer needs to be created and configured to allow the script host to communicate for both the app layers and operating system updates. Again, I use Windows 10 for this process but I’m assuming Windows 2016 will also work. The following steps can be used either on a new layer or new version.

Open up a PowerShell window as admin.

Enable WinRM to allow remote PowerShell commands

```powershell
enable-psremoting –force
```

Set Execution Policy to allow unsigned scripts

```powershell
Set-ExecutionPolicy Unrestricted
```

[Chocolatey](https://chocolatey.org/) is a command line based package manager that can either use a public or custom repository of [numerous applications](https://chocolatey.org/packages). Chances are if you have a publicly downloaded app a Choclately package exists. We will install Chocolately with the following command.

 [![logo square](/assets/images/content/sdk/logo_square.png)](https://chocolatey.org/)

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

The following modules need to be installed that assist in the update process. [PSWIndowsUpdate](https://www.powershellgallery.com/packages/PSWindowsUpdate) handles installing Windows updates from Powershell and [Autologon](https://www.powershellgallery.com/packages/Autologon) can enable the interactive auto login process after reboots that will be used for the update and install processes. Install both using the following commands.

```powershell
Find-module -Name PSWindowsUpdate  
Install-Module -Name PSWindowsUpdate  
Find-Module -Name Autologon  
Install-Module -Name Autologon
```

In my image I needed to disable the firewall for “domain”. Run the following to disable the firewall for the “Domain” connection type

```powershell
Set-NetFirewallProfile -Name "Domain" -Enabled False –Verbose
```

Before sealing the OS layer let’s make sure WinRM is accessible via the script host

```powershell
Test-WSMan -ComputerName ‘IPADDRESSOFVM’
```

Validate your admin credentials work. **USE THE LOCAL ADMIN ACCOUNT.**

```powershell
$cred = get-credential  
New-PSSession -ComputerName ‘IPADDRESS’ -Credential $cred
```

If the following commands run successfully, go ahead seal and finalize the layer

**You are now ready to start automating!**

## App Layer Creation and Updating

In this section, we will cover fully automating the app layer creation and update process using some common Chocolatey packages. The script works in the following way

1.  Checks for existing App Layer  

    1.  If App layer is found with “appname” it compares the latest layer version against the Chocolatey repository. Only if a new version is found will it create a new layer.
    2.  If no App layer is found it creates a new layer reflecting the latest Chocolatey version
2.  Gets IP for ELM created update VM using PowerCLI
3.  Waits for update VM to become available and waits for WinRM
4.  Once WinRM is available invokes chocolatey to install app
5.  Reboots once and enables auto login to clear any runonce processes
6.  Attempts to seal image with ELM bat commands
7.  If error found VM terminates
8.  Shutdown if successful
9.  Layer finalizes
10.  Begins process for next app
11.  Reports on layers after script completion

### The script

Let’s get started by opening up the **BuildAppLayer.ps1** file in ISE or Code (run as admin) and make changes to reflect your environment. Additional Choclatey packages can be found [here](https://chocolatey.org/packages).

For this example, we will be installing Notepad Plus Plus, Firefox, and Chrome

#### Run

When ready go ahead and run the script. You should start to see the activity within vCenter and process kick off. Grab a cup of coffee and wait for the layers to create! Once completed they can be added to an image. MAKE SURE TO TEST!!

![coffee](/assets/images/content/sdk/coffee.gif)

## OS Layer Windows Updates

In this section, we will cover a method to apply any recent Windows updates to a new OS version.  Since Windows Update is restricted from running in a remote PowerShell session it requires any scripts to be run locally from the VM.  To circumvent this restriction the script creates a scheduled task that launches on boot to check for an apply the updates.  It does the following.

1.  Creates a new ELM OS version layer
2.  Grabs IP for ELM update VM
3.  Waits for WinRM to become available
4.  Copies script to update VM that scheduled task will launch
5.  Creates a scheduled task on update VM to run on boot
6.  Enables auto logon so update VM logs in to start the process
7.  Reboots VM and script host waits for VM to shutdown
8.  On boot, update VM logs in and starts the scheduled task which runs local update script
9.  Update script uses PSWindowsUpdate to check for and apply updates
10.  Script checks for any needed reboots
    1.  Reboots if necessary and script reruns
11.  If no updates found or reboots needed the script runs ELM seal commands
    1.  If error occurs VM again reboots
12.  If no errors are encountered with seal commands update script deletes itself and VM shuts down
13.  script host then finalizes the layer

**Note: The way this process is currently configured it can cause reboot loops on the update VM.  If the update VM doesn't shutdown in 2 hours the script terminates.**
{: .notice--info}

### The Script

Let’s get started by opening up the **BuildOSRev.ps1** and **UpdateTask.ps1** files in ISE or Code (run as admin) and make changes to reflect your environment.  Make sure to edit **BOTH** files. 

#### Run

Once your ready go ahead and run **BuildOSRev.ps1** If everything works correctly you should see the running process in ELM and the script will continue to check status.

![downby](/assets/images/content/sdk/downby.gif)
