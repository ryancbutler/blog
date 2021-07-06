---
layout: single
title: Getting Started with the Citrix App Layering Cloud API Service
categories:
  - Citrix
  - App Layering
author: Ryan Butler
tags:
  - Unidesk
  - App Layering
  - SDK
  - REST
  - PowerShell
---

Last week Citrix [released](https://www.citrix.com/blogs/2019/10/22/improve-job-throughput-with-citrix-app-layering-automation-enhancements/) a "Tech Preview" of the App Layering API.  The API as a starting point has some basic functionality that customers have been wanting for a long time such as querying layers and resources. The Citrix documentation provides a JSON spec but I also imported into [Swagger](https://app.swaggerhub.com/apis-docs/ryan_c_butler/app-layering_layering_services_api) to quickly see what it offers.  The API is very different than other Citrix SDKs\\APIS in that it requires an on-premises appliance that communicates to the ELM appliance while the API calls are made over the internet.  This post will go over the basics of getting started and provide a script that provides some of the functionality.

![main](/assets/images/content/layeringapi/main.png)

## Agent Appliance Configuration

In order to gain access to the appliance and API, [you will need](https://developer.cloud.com/applayering/overview) both a Citrix Cloud account ID and a submitted [form](https://podio.com/webforms/23191783/1654530) requesting access.  Once the request is approved you will receive instructions on how to install the appliance and provide the needed Cloud account information.  Once the appliance is registered with Citrix cloud make note of the registration ID to be used later.

![agentid](/assets//images/content/layeringapi/agentid.png)

## Registering ELM Appliance with Agent

Once the appliance is registered with Citrix Cloud the appliance then needs to register the on-prem ELM appliance.  In order to register the appliance you will need to identify your endpoint URL.  There are currently three.

* [https://us.workstreams.cloud.com/services/Citrix.AppLayering/master/LayeringServices](https://us.workstreams.cloud.com/services/Citrix.AppLayering/master/LayeringServices)
* [https://eu.workstreams.cloud.com/services/Citrix.AppLayering/master/LayeringServices](https://eu.workstreams.cloud.com/services/Citrix.AppLayering/master/LayeringServices)
* [https://ap-s.workstreams.cloud.com/services/Citrix.AppLayering/master/LayeringServices](https://ap-s.workstreams.cloud.com/services/Citrix.AppLayering/master/LayeringServices)

Like all the Citrix Cloud APIs a token will need to be requested to do any further commands. In order to request a token your cloud id and secret will be used.  Again, the below code will be provided in a single script at the end of this post.  
  
```powershell
function Get-BearerToken {  
    param (  
        [Parameter(Mandatory = $true)]  
        [string] $customerId,  
        [Parameter(Mandatory = $true)]  
        [string] $clientId,  
        [Parameter(Mandatory = $true)]  
        [string] $secret  
    )  
    $requestUri = "https://trust.citrixworkspacesapi.net/$customerId/tokens/clients"  
    $headers = @{"Content-Type" = "application/json" }  
      
    $auth = @{  
        "ClientId"     = $clientId  
        "ClientSecret" = $secret  
    }
   
    $response = Invoke-RestMethod -Uri $requestUri -Method POST -Headers $headers -Body (ConvertTo-Json $auth)  
    return $response.token  
}
```

*This function will return a token to be used for all further commands.*

Now that we have our token we can register our ELM appliance with the API agent. In addition to the token you will also need the agent ID that we documented earlier, the IP or hostname of the ELM appliance and admin credentials.

```powershell
function Invoke-ApplianceReg {  
    param (  
        [Parameter(Mandatory = $true)]  
        [string] $customerId,  
        [Parameter(Mandatory = $true)]  
        [string] $token,  
        [Parameter(Mandatory = $true)]  
        [string] $url  
    )  
    $requestUri = $url + "/resources/"  
    $headers = @{  
        "Content-Type"      = "application/json; charset=utf-8"  
        "Authorization"     = "CWSAuth bearer=$token"  
        "Accept"            = "application/json;version=preview"  
        "Citrix-CustomerId" = $customerId  
    }  
    $body = @{
   
        "name"                = "Butler's Appliance"  
        "description"         = "A description of my App Layering appliance"  
        "type"                = "Appliance"  
        "resourceLocationId"  = "1333b292...."  
        "applianceConnection" = @{  
            "address" = "192.168.2.50"  
            "auth"    = @{  
                "credentials" = @{  
                    "username" = "administrator"  
                    "password" = "mypassword"  
                }  
            }  
        }  
    }  
    $response = Invoke-RestMethod -Uri $requestUri -Method POST -Headers $headers -Body ($body | ConvertTo-Json -Depth 5)
   
    return $response  
}
```

Make sure to adjust the script to reflect your information.  Once the script is run you should receive information about your registration.

![appliance](/assets/images/content/layeringapi/appliance.png)

## Sync Appliance

Once the ELM is registered it must be synced with the agent in order to receive the needed metadata such as layers and appliance info.

```powershell
function Invoke-ApplianceSync {  
    param (  
        [Parameter(Mandatory = $true)]  
        [string] $customerId,  
        [Parameter(Mandatory = $true)]  
        [string] $token,  
        [Parameter(Mandatory = $true)]  
        [string] $url,  
        [Parameter(Mandatory = $true)]  
        [string] $ApplianceID  
    )  
      
    $headers = @{  
        "Content-Type"      = "application/json; charset=utf-8"  
        "Authorization"     = "CWSAuth bearer=$token"  
        "Accept"            = "application/json;version=preview"  
        "Citrix-CustomerId" = $customerId  
    }
   
    $requestUri = $url + "/resources/" + $ApplianceID + "/`$sync?async=true"
   
    $response = Invoke-WebRequest -Uri $requestUri -Method POST -Headers $headers  
    return $response.headers.Location  
}
```

This will return a Job URL that can be queried to see the progress.  After the job is completed you will notice further info available and you are ready to start automating!

**Note: There is no automatic sync.**  If layers are updated on the ELM a sync should be run to update the API agent
{: .notice--warning}

![appliancemore](/assets/images/content/layeringapi/appliancemore.png)

## The Script

The script is available [here](https://github.com/ryancbutler/Citrix/tree/master/Cloud%20API%20Service) that includes the above-mentioned functions and a few others to get you started.