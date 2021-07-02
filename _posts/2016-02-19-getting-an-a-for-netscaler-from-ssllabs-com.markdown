---
layout: single
title: Scoring an A+ for Netscaler from SSLLABS with Powershell
joomla_id: 16
joomla_url: getting-an-a-for-netscaler-from-ssllabs-com
excerpt: Citrix released a [blog](https://www.citrix.com/blogs/2015/05/22/scoring-an-a-at-ssllabs-com-with-citrix-netscaler-the-sequel/) early summer of 2015 outlining steps to take to harden SSL virtual servers to receive an "A+" from [SSLLABS](https://www.ssllabs.com/ssltest).  While the steps are easy to follow and doesn't take a lot of time for one Netscaler instance it can be time consuming for multiple instances. I created the following script to automate the process for all Load Balanced Servers (SSL), Netscaler Gateways and Content Switches (SSL) found on a Netscaler.  If need be you can even harden the management ports.  Simply edit the switches to reflect your environment and run. The script doesn't require any snapins but does require PowerShell 3.0 or greater for REST. Please feel free to leave any feedback on github or the comments below.
date: 2016-02-19 22:51:34.000000000 +00:00
author: Ryan Butler
permalink: /citrix/netscaler/16-getting-an-a-for-netscaler-from-ssllabs-com
categories:
  - Citrix
  - Netscaler
tags:
  - ADC
  - Netscaler
---
# Updates

- **6-13-16**: Updated Script to reflect [Citrix blog](https://www.citrix.com/blogs/2016/06/09/scoring-an-a-at-ssllabs-com-with-citrix-netscaler-2016-update/) with updated ciphers
- **2-21-16**: Script now creates STS policy and enables Forward Secrecy resulting in A+ for all SSL VIPS!
{: .notice--info}

* * *
Citrix released a [blog](https://www.citrix.com/blogs/2015/05/22/scoring-an-a-at-ssllabs-com-with-citrix-netscaler-the-sequel/) early summer of 2015 outlining steps to take to harden SSL virtual servers to receive an "A+" from [SSLLABS](https://www.ssllabs.com/ssltest).  While the steps are easy to follow and doesn't take a lot of time for one Netscaler instance it can be time consuming for multiple instances. I created the following script to automate the process for all Load Balanced Servers (SSL), Netscaler Gateways and Content Switches (SSL) found on a Netscaler.  If need be you can even harden the management ports.  Simply edit the switches to reflect your environment and run. The script doesn't require any snapins but does require PowerShell 3.0 or greater for REST. Please feel free to leave any feedback on github or the comments below.

![SSLAPlus](/assets/images/content/SSLAPlus.png)

## Download

[SCRIPT FOUND HERE](https://github.com/ryancbutler/Citrix/blob/master/Netscaler/)
{: .notice--info}

Thanks to [Carl Stalhood](http://www.carlstalhood.com/netscaler-scripting/) for a great starting point on the Netscaler API portion!