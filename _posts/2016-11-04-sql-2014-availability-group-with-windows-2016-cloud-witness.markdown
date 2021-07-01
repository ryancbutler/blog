---
layout: single
toc: true
toc_sticky: true
title: SQL 2014 Availability Group with Windows 2016 Cloud Witness
joomla_id: 26
joomla_url: sql-2014-availability-group-with-windows-2016-cloud-witness
date: 2016-11-04 19:25:17.000000000 +00:00
author: Ryan Butler
permalink: /microsoft/26-sql-2014-availability-group-with-windows-2016-cloud-witness
categories: 
  - Microsoft
tags:
  - SQL
  - Clustering
  - Azure
---
## What this guide covers

This configuration will include 3 SQL replicas across two sites\subnets.  2 will be located in the primary location where 'automatic failover' will be configured.  This will allow the primary node to automatically fail over to the secondary node at the primary site much like a mirror does.  The third node will reside in the secondary location in the case of site failure and will require a manual failover.

![Drawing](/assets/images/content/sqlag/Drawing.png)

## Introducing Cloud Witness

In today's world most enterprises leverage a secondary site for disaster recovery in some fashion and believe it should be a no-brainer for SQL as well.  With SQL mirroring, this was relatively simple by splitting the mirror and the SQL witness across the two locations for a quick DR configuration. But, in the last couple of years Microsoft has been pushing users to SQL availability groups that would piggy-back off of Windows failover clustering.  

Windows clustering is much different from mirroring since it utilizes a quorum to keep the cluster up and running vs a dedicated witness server.  If the cluster loses enough members it will stop, causing SQL to stop running accross all nodes. In a single site scenario this would be fairly easy to protect by adding a fileshare witness that would allow for a single node failure.  The big limitation came in when adding a second site since the fileshare would live in one of the two physical sites.  And in a quorum configuration, if the site that included the fileshare witness would go down it would bring the entire cluster down!  

To protect quorum the fileshare needs to reside in a third location which many enterprises don't have readily available.  One possibility is to use the cloud to host the fileshare but this would require infrastructure such as a VPN tunnel back to the provider.  This requires a lot of planning and work for a tiny file but extremely critical to keep the DB up an running.  To resolve this headache Microsoft introduced 'Cloud Witness' in Server 2016 which allows the fileshare to reside in Azure, but without any of the other infrastructure.  Just throw in the Azure API key and Server 2016 will take of the rest for you! No VPN needed! This guide will go over the steps to deploy a Windows Server 2016 failover cluster along with a SQL Enterprise 2014 availability group.

## Requirements

*   3 Windows Standard or Datacenter 2016 Servers (2 will also work)
*   SQL 2014 Enterprise (2 will also work)
*   2 IPs per subnet (For this guide you will need 4 in addition to the server IPs)
*   Account that has access to create computer objects and modify domain permissions
*   Azure access
*   Service account to run SQL

## Configure Azure Access

To create the cloud witness an Azure account and API key is required.

1.  Login to your Azure portal
2.  Create a new 'storage account'.  Feel free to adjust to reflect environment.  
    ![azure 01](/assets/images/content/sqlag/azure-01.png)
3.  After the account is created copy the API key needed for the next steps  
    ![azure 02](/assets/images/content/sqlag/azure-02.png)

* * *

### Feature Install

1.  Let's start with installing the necessary features on each of the Windows 2016 servers  
    ![features 01](/assets/images/content/sqlag/features-01.png)
2.  Repeat this process on each server node

## Create Cluster

1.  Log on to the first node with domain admin or an account that has access to create computer objects
2.  From server manager launch the newly installed 'Failover Cluster Manager'  
    ![create 01](/assets/images/content/sqlag/create-01.png)
3.  Select 'Create Cluster'  
    ![create 02](/assets/images/content/sqlag/create-02.png)
4.  The wizard will now run  
    ![create 03](/assets/images/content/sqlag/create-03.png)
5.  Enter the names of the nodes that you would like to participate.  In the screenshot only two nodes are added but feel free to add the third. (I go over adding additional nodes in a later section)  
    ![create 04](/assets/images/content/sqlag/create-04.png)
6.  Run through the validation tests  
    ![create 05](/assets/images/content/sqlag/create-05.png)  
    ![create 06](/assets/images/content/sqlag/create-06.png)  
    ![create 07](/assets/images/content/sqlag/create-07.png)  
    ![create 08](/assets/images/content/sqlag/create-08.png)
7.  Enter an IP address that the cluster will use for each subnet.  And choose a name for the cluster (this will create a computer object).  
    ![create 009](/assets/images/content/sqlag/create-009.png)  
    ![create 10](/assets/images/content/sqlag/create-10.png)
8.  Continue to hit next and verify there are no errors.

###  Create Cloud Witness

1.  From 'Failover Cluster manager' launch 'Configure Cluster Quorum Settings...'  
    ![configure 01](/assets/images/content/sqlag/configure-01.png)
2.  The configuration wizard will start  
    ![configure 02](/assets/images/content/sqlag/configure-02.png)
3.  'Select the quorum witness'  
    ![configure 03](/assets/images/content/sqlag/configure-03.png)
4.  Configure a cloud witness  
    ![configure 04](/assets/images/content/sqlag/configure-04.png)
5.  Enter in the Azure account name and API key  
    ![configure 05](/assets/images/content/sqlag/configure-05.png) 
6.  Select next to finish the wizard  
    ![configure 06](/assets/images/content/sqlag/configure-06.png)
7.  Cluster is now configured!  You should see a new container within Azure.  
    ![azure 03](/assets/images/content/sqlag/azure-03.png)

* * *

### Install SQL 2014

SQL needs to be installed on each Windows 2016 Server

1.  Start the SQL 2014 Enterprise installer  
    ![installsql 01](/assets/images/content/sqlag/installsql-01.png)
2.  Most of the screens can be continued through.  On the feature selection select 'SQL Server Replication' and management tools  
    ![installsql 06](/assets/images/content/sqlag/installsql-06.png)  
    ![installsql 07](/assets/images/content/sqlag/installsql-07.png)
3.  Feel free to change the instance or leave default  
    ![installsql 08](/assets/images/content/sqlag/installsql-08.png)
4.  Configure the service account to run SQL along with adjusting service startup option  
    ![installsql 09](/assets/images/content/sqlag/installsql-09.png)
5.  Configure SQL access.  I chose mixed mode.  
    ![installsql 10](/assets/images/content/sqlag/installsql-10.png)
6.  Continue through the prompts

### Configure SQL Service

Adjust the SQL instance so it can join the cluster.

1.  Run 'SQL Server 2014 Configuration' from the start menu and select 'SQL Server Services'
2.  View the properties of 'SQL Server'  
    ![sqlsvc 01](/assets/images/content/sqlag/sqlsvc-01.png)
3.  Select the 'AlwaysOn High Availabilty' tab and place a check in 'Enable 'AlwaysOn Availability Groups' and hit OK  
    ![sqlsvc 02](/assets/images/content/sqlag/sqlsvc-02.png)
4.  Restart the SQL server service  
    ![sqlsvc 03](/assets/images/content/sqlag/sqlsvc-03.png)

### Create Database Backup

For this example I created a single database called 'testag' running only on the first node.  A backup is required to place in a availability group.

1.  Start the backup wizard on the database  
    ![sqlbackup 01](/assets/images/content/sqlag/sqlbackup-01.png)
2.  Run the backup with all defaults and location of your choice  
    ![sqlbackup 02](/assets/images/content/sqlag/sqlbackup-02.png)
3.  Verify backup is complete  
    ![sqlbackup 03](/assets/images/content/sqlag/sqlbackup-03.png)

### Create Temporary Windows Share

The availability group wizard needs a Windows share that is accessible to all nodes where it will copy the backup files needed while setting up the group.  This share is only used when the group is created.

1.  Create a folder on the first node or use an existing path that will have access.  I used 'C:\TempShare'  
    ![tempshare 01](/assets/images/content/sqlag/tempshare-01.png)  
    ![tempshare 02](/assets/images/content/sqlag/tempshare-02.png)  
    ![tempshare 03](/assets/images/content/sqlag/tempshare-03.png)

### Create SQL 2014 Availability Group

1.  Run the 'New Availability Group Wizard'  
    ![createag 01](/assets/images/content/sqlag/createag-01.png)
2.  Select the name for the availability group.  
    ![createag 02](/assets/images/content/sqlag/createag-02.png)
3.  Select the database to add to the group. (A backup must be created before this step.)  
    ![createag 03](/assets/images/content/sqlag/createag-03.png)
4.  Add replicas.  In this case I only have two replicas but again feel free to add the third.  Verify 'Synchronous' is selected.  
    ![createag 04](/assets/images/content/sqlag/createag-04.png)
5.  Select the share path created earlier  
    ![createag 05](/assets/images/content/sqlag/createag-05.png)  

6.  AG will now create.  
    ![createag 06](/assets/images/content/sqlag/createag-06.png)

* * *

A listener is virtual object that allows connections to a database running on a node\replica of an availability group.  The listener will dynamically point to the primary replica and is used in ODBC connections and connection strings of applications. A virtual IP will be required for each subnet used in the group   In the example below 'ag16listen' is the configured listener with virtual IPs for each subnet.

![list](/assets/images/content/sqlag/list.png)

### Delegate Cluster account in AD

In order for the Availability group to create a listener computer object  the cluster computer account needs rights to create computer objects.

1.  Locate the cluster computer object within AD.  For this example 'always16' is the computer account located in the LAB OU.
2.  Right click on the OU and select security and then advanced.  If this option is unavailable make sure 'advanced features' is enabled under the view menu.  
    ![delegate 00](/assets/images/content/sqlag/delegate-00.png)
3.  Select add to add the computer object  
    ![delegate 01](/assets/images/content/sqlag/delegate-01.png)
4.  Select a principal  
    ![delegate 02](/assets/images/content/sqlag/delegate-02.png)
5.  Adjust the object types to look for computer accounts as well  
    ![delegate 03](/assets/images/content/sqlag/delegate-03.png)
6.  Select Computers  
    ![delegate 04](/assets/images/content/sqlag/delegate-04.png)
7.  Type in the name of the cluster created earlier. ('$' is used to lookup a computer account)  
    ![delegate 05](/assets/images/content/sqlag/delegate-05.png)
8.  Now edit the permissions of the computer account  
    ![delegate 06](/assets/images/content/sqlag/delegate-06.png)
9.  Select 'Create Computer Objects' and keep all the other defaults  
    ![delegate 07](/assets/images/content/sqlag/delegate-07.png)

### Create Availability Group Listener

1.  Within SQL managment studio on the primary replica, run 'Add Listener'  
    ![aglis 01](/assets/images/content/sqlag/aglis-01.png)
2.  Enter the name you would like the computer object and DNS name of the listener to be (this will be created in AD)
3.  Add a unique IP address for each subnet and port.(Recommend sticking with the SQL default of 1433.)  
    ![aglis 03](/assets/images/content/sqlag/aglis-03.png)
4.  Hit OK and verify the process runs correctly.  Most of the issues encountered are around the AD delegation of the cluster computer account.

### DNS Considerations

By default all subnet IPs will have a DNS entry created which is no problem for newer applications since modern ODBC drivers will be able to figure out what IP to connect with dynamically.  However, this might cause issues for applications that utilize older connection methods.  For example Citrix XenDesktop can utilize the more modern connection driver but until recently Provisioning Services (<7.11) would have issues when resolving to the secondary replica in the other subnet.

This is the current NSLOOKUP which shows each AG listener IP appearing and will cause a round robin lookup.

![dns 01](/assets/images/content/sqlag/dns-01.png)

To resolve the round robin result we can run PowerShell commands to register only the 'active' subnet of the primary. And if a failover does occur it will register the new IP to DNS in the secondary subnet

1.  First get the 'network name' needed

    ```powershell
    Get-ClusterResource | where {$_.resourcetype -eq 'Network Name'}  
    ```
    ![dns 02](/assets/images/content/sqlag/dns-02.png)</pre>

2.  Next run the following commands reflecting of the name discovered in the above command

    ```powershell
    Get-ClusterResource "always16-ag_ag16listen"|Set-ClusterParameter -Name HostRecordTTL -Value 5
    Get-ClusterResource "always16-ag_ag16listen"|Set-ClusterParameter -Name RegisterAllProvidersIP -Value 0
    Stop-ClusterResource "always16-ag_ag16listen"
    Start-ClusterResource "always16-ag_ag16listen"
    Get-ClusterResource "always16-ag_ag16listen" |Update-ClusterNetworkNameResource
    ```


3.  Verify NSLOOKUP and only 1 IP should appear  
    ![dns 03](/assets/images/content/sqlag/dns-03.png)

* * *

Follow these steps to add an additonal replica to your availablity group

### Add node to failover cluster

First need to add the additonal node to the failover cluster.

1.  Launch 'failover cluster manager' from the primary node. and select 'Add Node'  
    ![addnode 01](/assets/images/content/sqlag/addnode-01.png)
2.  Enter the name of the additional node  
    ![addnode 001](/assets/images/content/sqlag/addnode-001.png)
3.  Continue to hit 'next' and verify successful add
4.  If the additional node is in another subnet the cluster will need an IP in that new subnet so continue on.  If the node is in the existing subnet skip to the "Add Replica to Availability Group" section.
5.  Go to properties of the cluster name under the 'Cluster Core Resources' section  
    ![addnode 002](/assets/images/content/sqlag/addnode-002.png)
6.  Select 'Add'  
    ![addnode 003](/assets/images/content/sqlag/addnode-003.png)
7.  Enter the IP in the new subnet  
    ![addnode 004](/assets/images/content/sqlag/addnode-004.png)
8.  Hit OK twice and agree to warning  
    ![addnode 005](/assets/images/content/sqlag/addnode-005.png)

### Add Additonal Subnet IP to AG Listener

The IP from the new subnet must exist on the listener before adding to the AG

1.  From SQL studio right click on the 'Availability Group Listener'
2.  Select 'Add'  
    ![addiplis 01](/assets/images/content/sqlag/addiplis-01.png)
3.  Enter a unique IP in the new subnet of the node  
    ![addiplis 02](/assets/images/content/sqlag/addiplis-02.png)
4.  Ht OK

### Add Replica to Availability Group

Now that the listener is configured and server member of the cluster it can be added to the AG

1.  From SQL studio right click on the Availability Group and select properties  
    ![addrep 001](/assets/images/content/sqlag/addrep-001.png)  
2.  Connect to the secondary replicas with service account and hit next  
    ![addrep 002](/assets/images/content/sqlag/addrep-002.png)
3.  Select 'Add Replica'  
    ![addrep 003](/assets/images/content/sqlag/addrep-003.png)  

4.  Enter in new replica server name and hit connect  

5.  Select Synchronous  
    ![addrep 004](/assets/images/content/sqlag/addrep-004.png)
6.  Enter in the network share used earlier in the process  
    ![addrep 005](/assets/images/content/sqlag/addrep-005.png)
7.  Verify 'validation' process is successful
8.  Continue with any prompts
9.  Replica should now appear 'Green'  
    ![addrep 006](/assets/images/content/sqlag/addrep-006.png)
