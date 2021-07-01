---
layout: single
title: Netscaler 12 Firmware Now Includes Python
joomla_id: 28
joomla_url: netscaler-12-adds-python-to-firmware
date: 2017-08-16 15:14:17.000000000 +00:00
author: Ryan Butler
permalink: /citrix/netscaler/28-netscaler-12-adds-python-to-firmware
categories:
  - Citrix
  - Netscaler
tags:
  - Python
  - ADC
---
While recently upgrading my lab Netscaler to version 12, I noticed Python was getting installed so thought I would do a little looksee to see what's available.

## What's Included?

Python is installed at **/var/python/bin/python** but isn't part of PATH

![path](/assets/images/content/python/path.png)

```
export PATH=$PATH:/var/python/bin
```

Now we can easily run Python without the entire path.

![python ver](/assets/images/content/python/python-ver.png)

Looks like the installed version is fairly old but it's nice to have a modern programming language included now!

Also the path includes [Ansible](https://www.ansible.com/) which is interesting and opens the doors for a lot of automation.  If you try to run Ansible it looks like Python is trying to run from a different path.

![ansible error](/assets/images/content/python/ansible-error.png)

Let's fix that with a softlink

```
ln -s /var/python/bin/python /usr/local/bin/python
```

And now we can run Ansible correctly.

![ansible ver](/assets/images/content/python/ansible-ver.png)

## More Possibilities

The current install also includes PIP which allows downloading and installing of packages.  For example we can download a quick bandwidth tester.

```
pip install speedtest-cli
```

![test](/assets/images/content/python/test.png)

And since the Python path exists within /var the packages remain even after reboot!  

## Conclusion

The Python install isn't all that user friendly and not exactly sure what the main purpose is since it takes a little work to get it to work.  Nonetheless, I'm excited to see what Citrix continues to add around this and think it's a great option for administrators to have available.
