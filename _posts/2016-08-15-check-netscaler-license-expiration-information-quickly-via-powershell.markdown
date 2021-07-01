---
layout: single
title: Check Netscaler License Expiration Information Quickly via Powershell
joomla_id: 23
joomla_url: check-netscaler-license-expiration-information-quickly-via-powershell
date: 2016-08-15 02:24:03.000000000 +00:00
author: Ryan Butler
permalink: /citrix/netscaler/23-check-netscaler-license-expiration-information-quickly-via-powershell
categories:
  - Citrix
  - Netscaler
tags:
  - ADC
  - License
  - PowerShell
  - Netscaler
---
## All I did was reboot the thing!

If you have been dealing with Netscaler for awhile chances are you have rebooted an instance only to find no one can connect but everything is pingable.  After trying to refresh your browser multiple times, you frantically login to the Netscaler management IP and discover all your VIPs down, features disabled and SSL certificates no longer listed. WTF!

If you have been bitten by this before you realize almost immediately that the platform license has expired causing the Netscaler to boot unlicensed.  This sounds pretty innocent right?  You just get a new key or license and apply to the Netscaler and you're back up and running? Wrong! The Netscaler not only disables all the features you were running on it, but it totally removes the SSL certificates.  That means all those certificate pairs your colleague added with those generic file names, the SSL certificates you linked 5 deep and bindings to your 25 VIPs are gone!  The thing to remember here is if I have a license that expired 90 days ago but I never rebooted, the Netscaler has been running 100% operational until today when I rebooted.  There isn't any sort of alerting from a Netscaler perspective to let you know that the device is close to expiration.  To add even more complexity to the issue there is no easy way to even see when the expiration date is.  There is no method to see this date from the GUI or the CLI currently.  The only way to see this expiration date is by looking at each and every license file located on each Netscaler (/nsconfig/license). Yes, you must look at each Netscaler individually even in an HA pair.

![licfile](/assets/images/content/licensefile/licfile.png)

## There has to be a better Way

While having to recover from an expired license after a recent lab upgrade I decided to take a stab at creating a method to help this situation so it doesn't happen again.  I came up with a script that scans through a Netscaler looking for license files and pulls the important date information and calculates expiration time left. Below is the script running against a Netscaler in my lab with multiple license files listing expiry information for each feature.

![mullic](/assets/images/content/licensefile/mullic.png)

Here is the script running against multiple Netscalers in an HA pair.

![mulns](/assets/images/content/licensefile/mulns.png) 

Here is how I ran it against multiple NSIPS for an example.

```powershell
$netscalers = @("192.168.1.38","192.168.1.32")  
$exps = $netscalers|foreach{.\get-nslicenseexp.ps1 -nsip $_}  
$exps|ft -AutoSize
```

Now I can finally see when a Netscaler is expiring without having to access all the license files indvidually.  I plan on using the script before my upgrade script  and creating a scheduled task to send me expiry alerts based on a threshhold of days left.  Please leave comments or ideas here or on Github!

Thanks to [Steve Noel](https://twitter.com/steve_noel) for helping me test!

## Download Locations

### Github

* [Normal](https://github.com/ryancbutler/Citrix/blob/master/Netscaler/get-nslicexp.ps1)
* [RAW](https://raw.githubusercontent.com/ryancbutler/Citrix/master/Netscaler/get-nslicexp.ps1)

### PSGallery Script (PowerShell 5.0 or greater)

```powershell
Find-Script -name Get-NSlicexp  
Save-Script -name Get-NSlicexp -path <path>
```
</pre>

### PSGallery Module (PowerShell 5.0 or greater)

Brandon allowed me to add my function to his awesome Netscaler module found [here](https://github.com/devblackops/NetScaler).
{: .notice--info}


```powershell
Find-Module -name Netscaler  
Install-Module -name Netscaler

Get-NSlicenseExpiration
```

