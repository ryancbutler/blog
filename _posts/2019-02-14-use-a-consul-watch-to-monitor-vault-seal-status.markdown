---
layout: single
title: Use a Consul Watch to Monitor Vault Seal Status
categories:
  - HashiCorp
toc: true
toc_sticky: true
permalink: /hashicorp/37-use-a-consul-watch-to-monitor-vault-seal-status
author: Ryan Butler
tags:
  - Consul
  - Vault
  - Slack
---
In order for a Vault node to be functional, it needs to be in an unsealed state which decrypts the encryption key used for decryption and encryption of secret data. If a Vault node is sealed no secret data can be retrieved until it's unsealed.  A node can become sealed for a variety of reasons such as if a node reboots after an OS update or the vault service restarts.  In this post, I'll go over how a "Consul Watch" can be used to monitor the Vault service (or any other service) and send a slack alert if found to be critical (sealed).

![consul watcher main](/assets/images/content/consul-watcher/consul_watcher-main.png)

* * *

## Getting Started

You'll need the following before starting.

1.  Vault installed and configured (using v1.0.3)
2.  Consul configured for Vault backend (using v1.4.2)
3.  [Slack WebHook URL](https://api.slack.com/incoming-webhooks)
4.  Requires "Requests" Python module (pip install requests)

## Consul Watch Overview

A [Consul Watch](https://www.consul.io/docs/agent/watches.html) can be configured to fire either a script file, HTTP request or STDOUT on the 'type' of watch configured. To create a simple onetime watch the following can be run from the command line on one of the Consul nodes.

```
consul watch -type checks -service vault
```

You'll see that the command outputs the health of the service located on every Vault node in JSON format to STDOUT. To take this a step further if you run..

```
consul watch -type checks -state critical
```

This will only output data if a service is found in a critical state which will be the basis of our Vault watch.  We only want to alert if the service is found to be in a critical state.

## Script

In order for the watch to actually do something if an event is encountered, I have created a script that will send updated service information to Slack.  This script needs to be modified with your slack URL and copied to the Consul server that will be running the watch. Remember Requests is required for the script!

<script src="https://gist.github.com/ryancbutler/8bd6ecbf12cee6e0398534fb197e1b6f.js" type="text/javascript"></script>

For this example, I have copied the script to /usr/bin.  Make sure that script can be executed (chmod +x checkservice.py).  The output to Slack using the WebHook URL will be something similar to...

![consul watcher](/assets/images/content/consul-watcher/consul_watcher.png)

## Watch

The simplest way to launch a watch is with the command line.  Now that the script is in place we can configure our watch to look for critical services, then fire the script if found.

```
consul watch -type checks -state critical /usr/bin/checkservice.py
```

You can now test the watch by restarting the Vault service from another terminal window.  Once the Vault service restarts it will come up as sealed which in turn reports to Consul as critical and the script fires.

```
service vault restart
```

You'll notice after launching the watch from the command line the output remains in the terminal which will stop the watch from functioning as soon as you log out. In order for the watch to run while logged out, Consul can run watches with config files that the Consul service will use in the background.

Before continuing let's make sure the watch is stopped (ctrl-c) and Vault is unsealed.

Within the Consul config directory create a JSON file (eg /etc/consul/consul.d/watch-critical.json).  If more watches are needed in the future you can simply add to this JSON or create a brand new file in the same format since the entire directory is parsed.

```
{  
 "watches": [  
 {  
   "type": "checks",  
   "state": "critical",  
   "handler_type": "script",  
   "args": ["/usr/bin/checkservice.py"]  
 }  
 ]  
}
```

To enable the watch the Consul service needs to be restart

```
service consul restart
```

To test the watch, restart the Vault service from one of the nodes.

```
service vault restart
```

## Closing

A Consul watch can be a quick way to monitor Consul services and a lot more with very little effort.  I do wish there was more capability of filtering when watches fire and can get noisy with a lot of watches.  I feel like watches can do a good job supplementing very specific Consul or Vault monitoring but doesn't come close to replace normal system monitoring.