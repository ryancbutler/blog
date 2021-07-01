---
layout: single
title: Send Pushbullet Note via VMware Orchestrator
joomla_id: 13
joomla_url: send-pushbullet-note-via-vmware-orchestrator
date: 2015-11-12 20:45:41.000000000 +00:00
author: Ryan Butler
permalink: /vmware/13-send-pushbullet-note-via-vmware-orchestrator
excerpt: "Pushbullet is a great tool to use for alerting of a variety
  of scenarios. If you haven't looked at it before I highly recommend it and
  beats the heck out of email alerts. For example if I wanted to be alerted
  of a completed workflow or even an error in a workflow Pushbullet is a perfect solution.
  Alerts can be sent to one or multiple devices all at once. By leveraging
  the HTTP-REST plugin with Orchestrator this becomes very easy. &nbsp;Lets get started
  with a simple example on creating a workflow than can be then utilized in others."
categories: 
  - VMware
  - vro
tags:
  - vro
  - pushbullet
toc: true
---


# Send Pushbullet Notes


[Pushbullet](https://www.pushbullet.com/) is a great tool to use for alerting of a variety of scenarios.  If you haven't looked at it before I highly recommend it and beats the heck out of email alerts.  For example if I wanted to be alerted of a completed workflow or even an error in a workflow Pushbullet is a perfect solution.  Alerts can be sent to one or multiple devices all at once.  By leveraging the HTTP-REST plugin with Orchestrator this becomes very easy.  Lets get started with a simple example on creating a workflow than can be then utilized in others.


![pblogo](/assets/images/content/pushbullet-push/pblogo.png)

## Get Token

You'll need your access token for Pushbullet that will be used for authentication.  You can find your token by logging into Pushbullet and going to "Settings\Account" and making note.

![](/assets/images/content/pushbullet-push/01-accesstoken.png)


## Add Rest Host

First we will need to add the Pushbullet API url so we can use it with the plugin.

![](/assets/images/content/pushbullet-push/02-addresthost.png) 

Feel free to use a different name.

![](/assets/images/content/pushbullet-push/03-addresthost.png)

![](/assets/images/content/pushbullet-push/04-addresthost.png)

![](/assets/images/content/pushbullet-push/05-resthost.png)

Enter the token you noted earlier and the password can be anything

![](/assets/images/content/pushbullet-push/06-resthost.png)

![](/assets/images/content/pushbullet-push/07-resthost.png)

## Add Rest Operation

Now that we have the Host we need to create the operation to send to the Pushbullet URL.

![](/assets/images/content/pushbullet-push/07-restop.png)

 ![](/assets/images/content/pushbullet-push/08-restop.png)

Select the host we created earlier

 ![](/assets/images/content/pushbullet-push/09-restop.png)

Now enter the information the operation we want to run

![](/assets/images/content/pushbullet-push/10-restop.png)

Select Submit when completed to create operation


## Create Workflow

Within Orchestrator create a new workflow

![](/assets/images/content/pushbullet-push/11-wf.png)

Now edit the workflow and lets add some inputs to use for the content of the message to send

![](/assets/images/content/pushbullet-push/12-wf.png)

We will add two inputs

![](/assets/images/content/pushbullet-push/13-wf.png)

![](/assets/images/content/pushbullet-push/14-wf.png)

![](/assets/images/content/pushbullet-push/15-wf.png)

 Now we need to add an attribute used for the message

![](/assets/images/content/pushbullet-push/16-wf.png)

That says outstringnote

![](/assets/images/content/pushbullet-push/17-wf.png)


## Sciptable Task

Now we will add a scriptable task to place the content in format to be used with pushbullet

![](/assets/images/content/pushbullet-push/18-script.png)

 ![](/assets/images/content/pushbullet-push/19-script.png)

We are going to bind our inputs we created earlier

![](/assets/images/content/pushbullet-push/20-script.png)

Now we will bind our output variable

![](/assets/images/content/pushbullet-push/21-script.png)

Now for the actual script we use to convert to a string to send to Pushbullet.

```javascript
var outstringnote = JSON.stringify(convertme);
```

![](/assets/images/content/pushbullet-push/22-script.png)

## Add REST Operation to Workflow

Now we will add the REST operation we created earlier

![](/assets/images/content/pushbullet-push/23-addwf.png)

![](/assets/images/content/pushbullet-push/24-addwf.png)

Your workflow should look like this so far

![](/assets/images/content/pushbullet-push/25-addwf-2.png)

Now select the "Setup" button in the upper right to setup the bindings.

![](/assets/images/content/pushbullet-push/26-addwf.png)

Select the operation we created earlier

![](/assets/images/content/pushbullet-push/27-addwf.png)

All the params can be "skipped" and select the value for `defaultContenttype`

![](/assets/images/content/pushbullet-push/28-addwf.png)

Enter application\json and hit ok

![](/assets/images/content/pushbullet-push/29-addwf.png)

And select promote


## Edit Workflow

Back on the schema tab select edit of the "Invoke a REST operation"

![](/assets/images/content/pushbullet-push/31-addwf.png)

Select the IN tab and select the content parameter

![](/assets/images/content/pushbullet-push/32-addwf.png)

Now select the variable we used in our script

![](/assets/images/content/pushbullet-push/33-addwf.png)

Now hit save and close your workflow.  If you receive a validation warning just hit view details and follow any recommendations.

![](/assets/images/content/pushbullet-push/34-save.png)


## Run Workflow

Now we are ready to run our workflow.

![](/assets/images/content/pushbullet-push/35-run.png)

![](/assets/images/content/pushbullet-push/36-run.png)

Now hopefully this was completed successfully.  If it didn't work check through the log of the workflow run for some details.
