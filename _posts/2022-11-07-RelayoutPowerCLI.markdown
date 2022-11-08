---
layout: single
title: Invoking vSAN Re-layout with PowerCLI 
categories:
  - VMware
author: Ryan Butler
tags:
  - vSphere
  - vSAN
  - PowerCLI
---
Skyline Health was reporting "vSAN object format health" alerts after upgrading a bunch of clusters to 7.x. Each cluster requires a "re-layout" to clear the alert. Kicking off the process is fairly straight forward just requires a lot of clicks.

![main](/assets/images/content/relayout/objecthealth.png){: .align-center}

I figured I could do this with PowerCLI but struggled with finding an example anywhere. After a good amount of time of trial and error I have the below working code to perform the process.

```powershell
#connect to vcenter
connect-viserver -server "myvsphere.domain.com"
#Get view #https://developer.vmware.com/docs/powercli/latest/vmware.vimautomation.storage/commands/get-vsanview/#Default
#Run Get-VSANView to get full list
$vsanobj = Get-VSANView -Id "VsanObjectSystem-vsan-cluster-object-system"

#get clusters to perform layout
$clusters = get-cluster
foreach($cluster in $clusters)
{
    write-host $cluster.name
    #invoke re-layout (async)
    $vsanobj.RelayoutObjects($cluster.id)
}
```
