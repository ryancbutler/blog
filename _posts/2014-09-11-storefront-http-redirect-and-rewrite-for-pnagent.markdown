---
layout: single
title: Storefront HTTP redirect and rewrite for PNAGENT
joomla_id: 8
joomla_url: storefront-http-redirect-and-rewrite-for-pnagent
date: 2014-09-11 02:30:22.000000000 +00:00
author: Ryan Butler
permalink: /citrix/storefront/8-storefront-http-redirect-and-rewrite-for-pnagent
excerpt: From time to time I run into clients that have very old thin clients but
  want to make the jump to Storefront. While Storefront does offer "Legacy PNAGENT"
  it only can be utilized using the base URL, which if you are using Netscaler Gateway
  it must be HTTPS. This can be a problem with old thin clients since they probably
  won't understand the newer SSL certs that are out there since they lack the ability
  to update root CAs. The only way for these devices to function is to utilize
  HTTP instead of HTTPS.
categories: 
  - Citrix
  - Storefront
tags:
  - Storefront
  - PNAGENT
---
From time to time I run into clients that have very old thin clients but want to make the jump to Storefront.  While Storefront does offer "Legacy PNAGENT" it only can be utilized using the base URL, which if you are using Netscaler Gateway it must be HTTPS. This can be a problem with old thin clients since they probably won't understand the newer SSL certs that are out there since they lack the ability to update root CAs. The only way for these devices to function is to utilize HTTP instead of HTTPS. 

Below I use the Netscaler rewrite function to edit the config.xml that PNAGENT uses by replacing HTTPS for HTTP and some other optional changes.  I apply this rewrite only to traffic for PNAGENT and continue to redirect to HTTPS for all other via policy.

Here are the CLI commands to create the LB server on the Netscaler.

### Create the LB Server

```add lb vserver lb_sf_httpres HTTP 192.168.2.16 80 -persistenceType NONE -cltTimeout 180```

### Rewrite to HTTP

Change HTTPS to HTTP in config.xml

```
add rewrite action changepnagent_https_to_http replace_all "http.res.body(100000)" "\"http://\"" -pattern "https://"

add rewrite policy pnagent_https_to_http "http.req.url.path.endswith(\"/PNAgent/config.xml\")" changepnagent_https_to_http

bind lb vserver lb_sf_httpres -policyName pnagent_https_to_http -priority 100 -gotoPriorityExpression NEXT -type RESPONSE 
```

### Enabled Password Save

Not sure if this actually works as it should but this allows to save the password on a PNAGENT connection

```
add rewrite action act_storefront_enable_password_save replace_all "http.res.body(100000)" "\"<EnableSavePassword>true</EnableSavePassword>\"" -pattern "<EnableSavePassword>false</EnableSavePassword>"

add rewrite policy pol_storefront_enable_password_save "http.req.url.path.endswith(\"/PNAgent/config.xml\")" act_storefront_enable_password_save

bind lb vserver lb_sf_httpres -policyName pol_storefront_enable_password_save -priority 110 -gotoPriorityExpression NEXT -type RESPONSE
```

### Add Prompt and SSO Options 

If Storefront has SSO enabled it also adds prompt (http://support.citrix.com/proddocs/topic/dws-storefront-21/dws-configure-pna-auth.html)

```
add rewrite action act_add_prompt_and_sso replace_all "http.res.body(100000)" "\"<LogonMethod>sson</LogonMethod><LogonMethod>prompt</LogonMethod>\"" -pattern "<LogonMethod>sson</LogonMethod>"

add rewrite policy pol_storefront_add_prompt "http.req.url.path.endswith(\"/PNAgent/config.xml\")" act_add_prompt_and_sso

bind lb vserver lb_sf_httpres -policyName pol_storefront_add_prompt -priority 120 -gotoPriorityExpression NEXT -type RESPONSE
```

### Responder

Used if a user just enters hostname in pnagent field since it will redirect to PNAGENT site since it defaults in HTTPS /Citrix/PNagent/config.xml

```
add responder action res_act_tostore redirect "\"http://storefront.mydomain.com/Citrix/Store/PNAgent/config.xml\"" -bypassSafetyCheck YES

add responder policy pol_http_pnagnet_to_store "HTTP.REQ.URL.CONTAINS(\"/Citrix/PNAgent/config.xml\")" res_act_tostore

bind lb vserver lb_sf_httpres -policyName pol_http_pnagnet_to_store -priority 100 -gotoPriorityExpression END -type REQUEST
```

### Redirect anything else to HTTPS

```
add responder action Responder_redirect redirect "\"https://mycitrix.lab.local/Citrix/MainWeb\"" -bypassSafetyCheck YES

add responder policy http "HTTP.REQ.URL.CONTAINS(\"/PNAgent/\").NOT" Responder_redirect

bind lb vserver lb_sf_httpres -policyName http -priority 110 -gotoPriorityExpression END -type REQUEST
```
