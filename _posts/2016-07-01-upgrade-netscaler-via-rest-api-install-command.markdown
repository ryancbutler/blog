---
layout: single
title: Upgrade Netscaler via REST API Install Command
joomla_id: 20
joomla_url: upgrade-netscaler-via-rest-api-install-command
date: 2016-07-01 16:19:09.000000000 +00:00
author: Ryan Butler
permalink: /citrix/netscaler/20-upgrade-netscaler-via-rest-api-install-command
categories:
  - Citrix
  - Netscaler
tags:
  - ADC
  - Netscaler
  - REST
  - PowerShell
---
With the recent release of Netscaler firmware 11.1 from Citrix brings a new NITRO API command called "install" which allows firmware upgrades from the API.  I got pretty excited when I saw this and decided to take a look since I always felt this would be a great feature to have.  This post goes through how it functions and includes a powershell script that uses the new functionality for future firmware releases.

![NITROAPI Install](/assets/images/content/nitroapiinstall/NITROAPI-Install.png)

## Functionality

The command works by containing a URL that the Netscaler will use to download the firmware and perform the upgrade.  Other arguments include enabling "callhome" ('l') and rebooting ('y') after upgrade completion.  A couple of notes to be aware of in my testing.

1. Since I want to verify the upgrade is successful I would rather wait to reboot the Netscaler in a seperate command
2. Once the command is sent to the Netscaler the **SNIP** is used to actually download the firmware not the **NSIP.** This will be important for firewall rules.

## Script

[DOWNLOAD HERE](http://bit.ly/29b3u1d)

The [script](http://bit.ly/29b3u1d) will upgrade a Netscaler instance (remember only 11.1 or greater) given a URL containing the firmware with included options for "callhome" and reboot

```powershell
./upgrade-ns.ps1 -nsip 10.1.1.2 -url "https://mywebserver/build-11.1-47.14_nc.tgz"
```

The above command will use the default `NSROOT\NSROOT` credentials and the provided url.  Uses the default values of disabling "callhome" and rebooting after upgrade.

```powershell
./upgrade-ns.ps1 -nsip 10.1.1.2 -url "https://mywebserver/build-11.1-47.14_nc.tgz" -adminaccount nsadmin -adminpassword "mysupersecretpassword" -callhome -noreboot
```

This command uses a different set of Netscaler credentials, enables callhome and does not reboot after upgrade completetion
