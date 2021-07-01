---
layout: single
title: 'Script to edit IIS 7.5 and Storefront for application initialization '
joomla_id: 11
joomla_url: script-to-edit-iis-7-5-and-storefront-for-application-initialization
date: 2014-10-07 21:39:02.000000000 +00:00
author: Ryan Butler
permalink: /citrix/storefront/11-script-to-edit-iis-7-5-and-storefront-for-application-initialization
excerpt: "Another powershell script to adjust XML\\config files for storefront."
categories:
  - Citrix 
  - Storefront
tags:
  - XML
  - IIS
---
Another powershell script to adjust XML config files for storefront.  This time it's for IIS 7.5 "application Initialization" which loads services before someone accesses the site. The script completes the changes mentioned in [CTX137400](http://support.citrix.com/article/CTX137400)

**NOTE: Must be run as administrator**
{: .notice--info}

[Download Script Here](https://github.com/ryancbutler/Citrix/blob/master/Powershell/Storefront/SFApplicationInitialization.ps1)
