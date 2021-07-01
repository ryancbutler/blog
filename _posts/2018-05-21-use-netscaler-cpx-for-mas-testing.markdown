---
layout: single
title: Use Netscaler CPX for MAS Testing
joomla_id: 32
joomla_url: use-netscaler-cpx-for-mas-testing
date: 2018-05-21 22:17:06.000000000 +00:00
author: Ryan Butler
permalink: /citrix/netscaler/32-use-netscaler-cpx-for-mas-testing
categories:
  - Citrix
  - Netscaler
tags:
  - ADC
  - CPX
  - MAS
  - Docker
---
After Synergy this year I watched a great presentation by [Esther Barthel](https://twitter.com/virtuEs_IT) and [Carsten Bruns](https://twitter.com/carstenbruns) ([SYN220](https://www.youtube.com/watch?v=pcz7jQUxdg4)) where they covered MAS Stylebooks and Configuration Jobs.  After getting more and more comfortable with the playbooks they graciously provided, I wanted to create my own and needed a good test environment but didn't want to impact my VPX.  I could have deployed an additional VPX but figured CPX would be a good candidate since it can quickly be reset for testing.  While testing with Docker, I noticed each time my CPX container was shut down it was removed from MAS since it went unreachable.  I did some research and found that a CPX container could be registered with MAS saving a lot of time not having to re-register each time.  In this post, I'll cover how to use Docker with Docker-compose to deploy a CPX container and automatically register with MAS.

![cpx](/assets/images/content/cpxmas/cpx.png)

## What you'll need

To get started you'll need the following.

*   [Docker Host](https://docs.docker.com/install/) (can be a local PC)
*   [Docker-Compose](https://docs.docker.com/compose/install/)
*   [Citrix MAS Configured](https://docs.citrix.com/en-us/netscaler-mas/12/deploy-netscaler-mas.html)

## Docker-Compose File

Once everything is configured, login to your Docker host and create a new YML file and copy the contents below.  For this post it's called **cpx.yml**.

### Ports

In this example port **8888** will be used for any VIPs deployed via Stylebook.  The other ports listed are used to map the NSIP and allow remote access.  If you plan on adding additional services you will need the ports listed here.

### Environment

You'll want to adjust these variables to reflect your MAS and Docker environment.  The MAS SSL fingerprint can be viewed at **System\System Administration\View SSL Certificate.**

![fingerprint](/assets/images/content/cpxmas/fingerprint.png)

## Running CPX

Once the YML file is updated you are ready to launch the CPX.  Simply run.

`docker-compose -f cpx.yml up -d`

This uses the YML file that you created and runs in the background (-d) once launched.  If everything is running correctly you should see the CPX registered in MAS under **Networks\Instances\Netscaler CPX**

## Test Stylebook

After the CPX is registered you are now ready to launch against a Stylebook.   When needing an IP for a VIP use **127.0.0.1** and the port (8888) you set in the YML file.

![stylebook](/assets/images/content/cpxmas/stylebook.png)

Your CPX should now be listed in the Target Instances to apply to.  Once the Stylebook is applied you should see the configuration applied within MAS.

![webconfig](/assets/images/content/cpxmas/webconfig.png)

To access the VIP remotely use the docker host IP and port you set in the YMl file.  For example [http://192.168.5.5:8888](http://192.168.5.5:8888)

## Stopping CPX

To stop and remove the CPX container run

`docker-compose -f cpx.yml down`

After a short period of time the CPX instance will also be removed from MAS