---
layout: single
title: Deploy Citrix Server VDA on Azure with Packer
joomla_id: 33
joomla_url: deploy-citrix-server-vda-on-azure-with-packer
date: 2018-08-15 21:01:08.000000000 +00:00
author: Ryan Butler
permalink: /microsoft/azure/33-deploy-citrix-server-vda-on-azure-with-packer
categories:
  - Microsoft
  - Azure
tags:
  - Packer
  - HashiCorp
  - CVAD
  - VDA
---
![packer](/assets/images/content/packer/packer.png)

In this post, I'll cover how to deploy a fully automated Citrix Windows 2016 VDA on Azure using [Packer](https://www.packer.io/). Packer is a very lightweight, open sourced tool to deploy identical images or templates from code.  This process is designed to quickly deploy a Citrix VDA to be used for a POC or test environment.  To get started you'll need the following.

* Azure account along with your subscription ID. Use the following to show the ID.

    ```powershell
    Login-AzureRmAccount
    ```

* Azure Powershell module installed

    ```powershell
    Install-Module -Name AzureRM
    ```

* The [Server OS Virtual Delivery Agent](https://www.citrix.com/downloads/xenapp-and-xendesktop/product-software/xenapp-and-xendesktop-718.html) downloaded and placed on a web server.  (This can be stored as an Azure block blob or another web server accessible via the internet)
* Packer downloaded and installed to your machine ( I use [chocolatey](https://chocolatey.org/packages/packer))

    ```powershell
    choco install packer
    ```

* [Grab the needed files from my GitHub](https://github.com/ryancbutler/azurectxvda)

    ```
    git clone https://github.com/ryancbutler/azurectxvda.git
    ```

## What it does

Packer will use the included template to perform the following during image creation

1. Deploy Windows 2016 VM with Public IP
2. Install Windows features and roles
3. Reboot
4. Install PowerShell Modules
5. Starts the VDA Install
6. Reboot
7. Resumes the VDA Install
8. Reboot
9. Installs apps via chocolatey
10. Converts to useable VHD for Citrix Machine Catalog

## Configure Azure for Packer Access

Packer will need an "App Registration" in order to have the appropriate Azure access. To streamline the process I modified [a script](http://www.tomsitpro.com/articles/build-custom-soe-on-azure-with-packer,1-3649.html) that creates the required access and outputs the values needed for Packer.

1. From the downloaded files open up **GetNeeded.ps1**
2. Edit the parameters to match your environment including subscription ID
3. Save and run PS1 file
4. If the script runs correctly it will output the needed values for the Packer template.

## Edit JSON Packer Template (**windows2016vda.json)**

Template file that Packer uses to create the image.

1. Using the values from above output edit **windows2016vda.json**
2. Save file

## Edit Citrix VDA Install Script (**ctxvda.ps1)**

This script installs the Citrix VDA

1. Open **ctxvda.ps1**and add the URL of your Citrix VDA to line 2\. (Make sure to include the trailing /)
2. Edit **filename** on line 1 to reflect the uploaded filename
3. Edit the **controllers** argument on line 3 to reflect your environment
4. Edit any other arguments on line 3 you may need.
5. Save file

## Optional Steps

### Edit PowerShell Modules (modules.ps1)

The modules this script installs are not really used except to install chocolatey. Feel free to edit.

### Edit Windows Features (features.ps1)

**features.ps1** contains Windows roles and features that get installed during the process.  Currently installs.

* RDS-RD-Server
* NET-Framework-45-Core
* Remote-Assistance
* Telnet-Client
* RSAT-DNS-Server
* RSAT-DHCP
* RSAT-AD-Tools

### Edit Applications Installed (**choco.ps1)**

**choco.ps1** contains applications that get installed during the process with Choclately.  The script installs

* Google Chrome
* Visual Studio Code
* 7-Zip
* Notepad ++
* Putty
* Git

### Run Packer

Once the files are edited and saved, go to the files directory either with Powershell or CMD and run

```
packer build .\windows2016vda.json
```

This will now go through the Packer process of building the VM and converting to a VHD file to create a Machine Catalog.  Make sure to validate everything runs correctly.

## Copy VHD for Studio access

While creating the Machine Catalog, Citrix Studio can only access containers at the root level and since Packer places the VHD files within sub-folders the file will need to be copied or moved to the root.

Packer creates the VHD in the following format

**/system/Microsoft.Compute/Images/*<capture_container_name>*/*<capture_name_prefix>*.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd**

Studio can only access

***<storage_account>*/system**

There are multiple different ways of copying the VHD but I recommend [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/).  Once copied to the root the VHD can be accessible from Studio.
