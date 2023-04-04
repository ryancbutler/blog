---
layout: single
title: Send Citrix Director alerts to Slack via Octoblu
joomla_id: 24
joomla_url: send-citrix-director-alerts-to-slack-via-octoblu
date: 2016-09-16 20:56:34.000000000 +00:00
author: Ryan Butler
permalink: /citrix/octoblu/24-send-citrix-director-alerts-to-slack-via-octoblu
excerpt: "Citrix released XenDesktop 7.11 this week which brings a fantastic feature where alerts can be sent to webhooks. Director now allows you to create alerts based on a variety of metrics and then send those alerts to a specific webhook when the threshold is met. "
categories:
  - Citrix
  - Octoblu
tags:
  - Alerts
  - Slack
  - Director
  - Octoblu
---
## Getting Hooked

Citrix released XenDesktop 7.11 this week which brings a fantastic feature where alerts can be sent to webhooks.  Director now allows you to create alerts based on a variety of metrics and then send those alerts to a specific webhook when the threshold is met.  (Who needs email alerts??)  The main use documented by Citrix is for Octoblu, but I don't see why it wouldn't work for anything that has some REST capability like VMware Orchestrator, AWS Lambda, RES Automation...  This post will go over a simple alert based on CPU and then send that alert to a Slack channel with information on the alert.

![main2](/assets/images/content/slackdirector/main2.png)

### What you'll need

* [Octoblu](https://octoblu.com/) account
* [Slack](https://slack.com/) account and access to integrations
* Access to Citrix director and DDC with XenDesktop 7.11 installed  

## Octoblu

* Let's start with the basic configuration of the Octoblu workflow and grabbing the needed webhook URL used with director.  Lay out the workflow in a similar manner to the screenshot below.  I usually throw the debug switch on everything for troubleshooting.
* ![title2](/assets/images/content/slackdirector/title2.png)

1 Trigger  
1 Template  
1 http Post

From the trigger ('From Director' in the screenshot) we will need to grab the "HTTP POST" web hook. It will look something like **[https://triggers.octoblu.com/v2/flows/UIDS](https://triggers.octoblu.com/v2/flows/UIDS)** copy this somewhere to remember.

## Citrix Director

Now we will configure our alert and then configure it to send information to our Octoblu webhook.

Login into director and select alerts from the top and then "Citrix Alerts Policy" tab.  For this demonstration I created a delivery group alert and assigned to test group but can be customized anyway you see fit.  Make sure to note the name of the alert (e.g. CPUALERT)

![alert](/assets/images/content/slackdirector/alert.png)

### Enable webhook

Now we will tell Director to call the webhook with this alert.  From the DDC launch Powershell.

Load up the Citrix snapins.

```powershell
add-pssnapin citrix*
```

Next we need the UID of our alert we just created with the following command.

```powershell
Get-MonitorNotificationPolicy
```

![get1](/assets/images/content/slackdirector/get1.png)

Next we use that UID to assign the Octoblu webhook URL

```powershell
Set-MonitorNotificationPolicy -UID _YOURUID_ -webhook _WEBHOOKURL_
```

![set](/assets/images/content/slackdirector/set.png)

Now let's verify the webhook is set

![get](/assets/images/content/slackdirector/get.png)

##  Slack Webhook

Slack also has incoming webhooks which allows you to send messages to a specific channel.  We will need this URL for the Octoblu workflow.

<span style="line-height: 2;">Login to Slack and go to settings and select "Configure Apps".</span>

![configure](/assets/images/content/slackdirector/configure.png)

Next select "Custom Integrations" and "Incoming WebHooks"

![incom](/assets/images/content/slackdirector/incom.png)

Select the channel you would like to use for this alert.  For this example we will use the **#General** channel and select "Add Incoming WebHooks Integration"

![getweb](/assets/images/content/slackdirector/getweb.png)

Copy the webhook Slack generates

![genweb](/assets/images/content/slackdirector/genweb.png)

## Complete Octoblu flow

Now we have the needed information to finish our flow.  Let's edit our template to make a message we want to send to slack.

![temp1](/assets/images/content/slackdirector/temp1.png)

My message is.

```
OH NO! CPU IS RUNNING AT {{msg.data.Value}}% AND {{msg.data.Priority}}!
```

This can be customized to anything you want just use the variables below.  I wish it could be more specific as to where the alert was triggered from for some of the larger scopes such as 'site' or large 'delivery groups'.  Right now, you can only see the alert for the whole alert scope and no information as to what specific VDA is causing triggering it.  Here are some of the other variables that come in the JSON body to use in the message. Use them in the message with the `{{ "{{ " }}}}` such as `{{msg.data.Condition}}`.

```
Priority = Critical or Warning  
Target = What the alert is assigned to  
Condition = Alert type  
Value = Metric value of alert  
Timestamp = When it was sent  
PolicyName = Alert name
```

Next we need to edit the "http POST" to use our message and send to the Slack webHook

![slackweb](/assets/images/content/slackdirector/slackweb.png)

Adjust the keys to use our message and few other modifications.  You can find more settings [here](https://api.slack.com/incoming-webhooks) from Slack.

![key](/assets/images/content/slackdirector/key.png)

Make sure to save the flow and approve any Octoblu permissions that are needed from the "Permissions Inspector".

Now if the CPU hits the threshhold you should receive an alert to your slack channel!  I used good ol [PRIME95](http://www.mersenne.org/download/) to spike the CPU for the test.

## ![sentalert1](/assets/images/content/slackdirector/sentalert1.png)
