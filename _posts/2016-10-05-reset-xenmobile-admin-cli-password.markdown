---
layout: single
title: Reset XenMobile admin CLI password
joomla_id: 25
joomla_url: reset-xenmobile-admin-cli-password
date: 2016-10-05 22:39:54.000000000 +00:00
author: Ryan Butler
permalink: /citrix/25-reset-xenmobile-admin-cli-password
excerpt: "This guide will go through the steps in resetting the local CLI admin account on a XenMobile 10.x virtual appliance. to complete"
categories: 
  - Citrix
tags:
  - XenMobile
---
This guide will go through the steps in resetting the local CLI admin account on a XenMobile 10.x virtual appliance.

**Please be warned this is not approved by Citrix and i'm not responsible for any issues this causes!**
{: .notice--warning}

### What's needed

*   Linux bootable media (I used an Ubuntu desktop ISO)
*   Access to VM console of XenMobile appliance
*   You will need to reboot each node to complete

## Password Reset Process

1.  Power down the XenMobile Node and "edit settings" of the VM.
2.  Add a CD\DVD to the VM and mount the Ubunutu desktop ISO  
    ![addcd](/assets/images/content/resetxm/addcd.png)  
    ![diskoptions](/assets/images/content/resetxm/diskoptions.png)
3.  Adjust the VM options to boot to the BIOS so we can change the boot order  
    ![bootoptions](/assets/images/content/resetxm/bootoptions.png)
4.  Adjust the options to boot to CD first  
    ![bootorder](/assets/images/content/resetxm/bootorder.png)
5.  Now Power up the VM and select "Try Ubuntu" to boot to the desktop from the ISO. Don't install.
6.  Once the desktop is loaded open "Terminal" from the menu on the left
7.  Once in the terminal we need to run the following commands as root so we use sudo..  

  ```bash
  sudo -i
  ```  
  ![sudo](/assets/images/content/resetxm/sudo.png)

8.  Mount the XenMobile appliance OS partition to the /tmp folder

  ```bash
  mount /dev/sda3 /tmp 
  ```

  ![mount](/assets/images/content/resetxm/mount.png)

9.  Run chroot from the mounted location

  ```bash
  chroot /tmp
  ```

  ![chroot](/assets/images/content/resetxm/chroot.png)

10.  If you aren't sure what the 'admin' account is named you can look at the shadow file of the appliance to get a listing.  You can see from the screenshot that **admin** is listed.  **DO NOT RESET ANY OF THE OTHER ACCOUNTS.**
    
  ```bash
  cat /etc/shadow
  ```

  ![users](/assets/images/content/resetxm/users.png)

1.  Now that we know our account we can reset it.  (This command will blank out the password.  You won't be able to set it here.)
    
    ```bash
    passwd admin
    ```
    ![passwd](/assets/images/content/resetxm/passwd.png)

12.  Unmount the file system
    
  ```bash
  umount /tmp
  ```

  ![unmount](/assets/images/content/resetxm/unmount.png)
13.  Shutdown the box from Ubuntu and remove the CD\DVD drive.

  ```bash
  shutdown -h now
  ```

  ![removecd](/assets/images/content/resetxm/removecd.png)
14.  You should now be able to login to the appliance from the console with the account you just reset with no password
15.  Once in make sure to set an actual password!  
    ![systemmenu](/assets/images/content/resetxm/systemmenu.png)
16.  Repeat same process on other XenMobile nodes
