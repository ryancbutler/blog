---
layout: single
title: 'Redirect to Full Store Web Path with Netscaler '
joomla_id: 12
joomla_url: redirect-to-full-store-web-path-with-netscaler
date: 2014-11-14 15:54:08.000000000 +00:00
author: Ryan Butler
permalink: /citrix/netscaler/12-redirect-to-full-store-web-path-with-netscaler
excerpt: "I hate having to edit single files on multiple servers since it can cause
  consistency issues and a pain if you need to make changes."
categories:
  - Citrix 
  - Netscaler
tags:
  - ADC
  - rewrite
  - policy
  - Netscaler
---
I hate having to edit single files on multiple servers since it can cause consistency issues and a pain if you need to make changes.  To redirect users to the full Storefront URL it took editing\creating a javascript snippet pointing to the full Storefront web URL.  By using the Netscaler for this process saves the time needed to touch each server and one less thing to worry about. 

```
add rewrite action rw_action_storefront replace HTTP.REQ.URL "\"/Citrix/StoreWeb\""
add rewrite policy rw_pol_storefront "HTTP.REQ.URL.EQ(\"/\")" rw_action_storefront
```
