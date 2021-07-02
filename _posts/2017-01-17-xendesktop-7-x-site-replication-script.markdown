---
layout: single
title: XenDesktop 7.x Site Replication Script
joomla_id: 27
joomla_url: xendesktop-7-x-site-replication-script
date: 2017-01-17 04:47:14.000000000 +00:00
author: Ryan Butler
permalink: /citrix/27-xendesktop-7-x-site-replication-script
categories: 
  - Citrix
tags:
  - PowerShell
---
![synctitle](/assets/images/content/syncyouverymuch/synctitle.png)
If you have ever designed or implemented a mutli-site environment with Citrix XenDesktop 7.6 LTSR or greater chances are you deployed separate dedicated sites. Yes, there is the possibility of stretching the XenDesktop database across physical locations, utilizing connection leasing or even the newly introduced local host cache in 7.12. But, the problem with this architecture is it brings in strict latency and or environment size requirements that isn't always possible or cost effective. 

Most of the time organizations end up with multiple XenDesktop sites and tasked with managing multiple sites totally independent from each other, which would include the separate management of user access, delivery groups, published applications and published desktops. This can create major consistency issues from one site to the next and a huge headache with very dynamic environments for any Citrix admin.

To hopefully help with the problems encountered with this design I created a PowerShell script utilizing the Citrix XenDesktop PowerShell SDK that will export XenDesktop site data including published applications, desktops and a variety of other settings (see link below for full listing) from one site and import to another. The script was designed so data will be exported from a main site and replicated to one or many secondary sites (think of robocopy for XD). The script could easily be tied to a Windows Task Schedule or added to any workflow.

[DOWNLOAD SCRIPT HERE](https://github.com/ryancbutler/XDReplicate)
{: .notice--info}

Please let me know of any comments or questions!
