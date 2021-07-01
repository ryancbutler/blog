---
layout: single
title: vRA Update Errors When Reducing Appliance CPU and RAM
joomla_id: 17
joomla_url: vra-appliance-update-errors
date: 2016-04-30 03:05:55.000000000 +00:00
author: Ryan Butler
permalink: /vmware/17-vra-appliance-update-errors
categories: 
  - VMware
  - VRO
tags:
  - vro
---
If you run a lab or a small vRealize Automation environment chances are you reduced the appliance VM resources from 4 vCPU and 18 GB of RAM to something smaller.  For example in my lab I reduced to 2 vCPU and 8 GB of RAM which works just fine.  Problem is when I attempt an upgrade for the vRA appliance from **7.0.0.1460 Build 3311738** to **7.0.1.150 Build 3622989** I get the below error.

![vra error](/assets/images/content/vra/vra-error.png)

<span style="text-decoration: underline;">Number of processing units should be increased to 4 (4 virtual CPU x 1 cores). - Memory should be increased to 18 GB.</span>

I found the script with these pre-install checks and edited to reflect my setup here: **/etc/bootstrap/preupdate.d/00-00-va-resources-check**

**LINE 14** where I adjusted the vCPU

<pre>
# Checking available processing units  
proc=$(nproc)  
if [ "${proc}" -lt <span style="background-color: #ffff00;">2</span> ]; then  
error=$' - Number of processing units should be increased to 4 (4 virtual CPU x 1 cores).\n'  
fi
</pre>

**LINE 20** where I adjusted the RAM

<pre>
# Checking for physical operating memory - decrease the actual reported MemTotal with around 1% to allow some tolerance
mem=$(grep MemTotal /proc/meminfo | sed 's/[^0-9]//g')
if [ ${mem} -lt <span style="background-color: #ffff00;">8000</span> ]; then
error=${error} - Memory should be increased to 18 GB.\n'
fi
</pre>

I was then able to successfully update

```bash
vamicli update --check  
vamicli update --install latest --accepteula</pre>
```