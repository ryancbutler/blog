---
layout: single
title: Speed up Storefront with ASPNET.config change script
joomla_id: 10
joomla_url: speed-up-storefront-with-aspnet-config-change-script
date: 2014-10-03 21:26:49.000000000 +00:00
author: Ryan Butler
permalink: /citrix/storefront/10-speed-up-storefront-with-aspnet-config-change-script
excerpt: "I got sick of having to manually edit Aspnet.config files to disable
  signature&nbsp;checking so Storefront would load faster. I created a script
  that once run from a single storefront server will pull the list of servers in the
  Storefront cluster and quickly look for Aspnet.config files remotely"
categories:
    - Citrix
    - Storefront
tags:
    - IIS
    - Storefront
---
I got sick of having to manually edit *Aspnet.config* files to disable signature checking so Storefront would load faster.  I created a script that once run from a single storefront server will pull the list of servers in the Storefront cluster and quickly look for *Aspnet.config* files remotely.  If the file is found not to have the tweak it will back the file up, add the `generatePublisherEvidence` line and restart IIS.

*   **Uses new Powershell modules**
*   **Disables .NET signature checking**
*   **Enables pool sockets**
*   **Disables netbios via WMI**

**NOTE: MUST BE RUN AS ADMINISTRATOR.**
{: .notice--info}

[Download from Github](https://github.com/ryancbutler/Citrix/blob/master/Powershell/Storefront/SFTweaks.ps1)