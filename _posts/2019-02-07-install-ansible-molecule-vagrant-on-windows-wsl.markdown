---
layout: single
title: Install Ansible, Molecule, Vagrant on Windows WSL
categories:
  - Ansible
author: Ryan Butler
permalink: /ansible/36-install-ansible-molecule-vagrant-on-windows-wsl
toc: true
toc_sticky: true
tags:
  - Ansible
  - WSL
  - Molecule
---
Ansible is a really cool and very popular config management (and a lot more!) tool but sadly the control plane only runs on Linux based systems.  As primarily a Windows user I wanted to see if it would function in the "newish" WSL environment and after a lot of trial and error found that it works great!  In this quick post I go over how to install Ansible for config management, Molecule to test roles, Vagrant to run the tests all while running on Windows WSL.

![main](/assets/images/content/molecule/main.png){:height="75%" width="75%"}

* * *

## Windows Setup

1.  [Install Virtual Box](https://www.virtualbox.org/wiki/Downloads)
2.  [Install Ubuntu WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
3.  Reboot computer

## WSL Setup

1.  Run Ubuntu from the start menu and upgrade all packages

  ```
    sudo apt update && apt upgrade -y
  ```

2.  Install Ansible

    ```
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt install ansible
    ```

3.  Install Python pip

    ```
    sudo apt-get install -y python-pip libssl-dev
    ```

4.  Install Pip packages

    ```
    sudo pip install molecule
    sudo pip install python-vagrant
    ```

5.  Set ENV variable so vagrant knows its running in WSL

    ```
    export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
    ```

6.  Install Vagrant and plugins

    ```
    sudo apt install vagrant  
    sudo vagrant plugin install vagrant-libvirt
    ```

7.  In order for WSL to know where Virtual box is installed some additional paths need to be added  

    ```
    export PATH=$PATH:/mnt/c/Windows/System32
    export PATH="$PATH:/mnt/c/Program Files/Oracle/VirtualBox"
    ```

8.  If you want to prevent the need to run the above **EXPORT** commands on every login add the following to the bottom of ~/.bashrc

    ```
    export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"  
    export PATH=$PATH:/mnt/c/Windows/System32  
    export PATH="$PATH:/mnt/c/Program Files/Oracle/VirtualBox"
    ```

9.  You are now ready to run Ansible and Molecule with WSL!

## Molecule Gotchas

By default, Molecule uses the ubuntu/xenial64 box for test runs.  During my initial testing I kept encountering an error like the one below. 

![error](/assets/images/content/molecule/error.png)

After inspecting the **/tmp/molecule/base-test/default/vagrant-instance.err** log I found this.

```
Stderr: VBoxManage.exe: error: RawFile#0 failed to create the raw output file /usr/local/lib/python2.7/dist-packages/molecule/provisioner/ansible/playbooks/vagrant/ubuntu-xenial-16.04-cloudimg-console.log (VERR_PATH_NOT_FOUND)  
VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component ConsoleWrap, interface IConsole
```

After some searching, I found the following workaround for **molecule.yml** and everything started to run correctly.

```
"customize [ 'modifyvm', :id, '--uartmode1', 'disconnected' ]"
```

![workaround](/assets/images/content/molecule/workaround.png)