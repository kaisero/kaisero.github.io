---
layout: post
title:  "Troubleshooting Firepower User Identity"
date:   2017-01-14
tags: [firepower, user identity, troubleshooting]
---

Troubleshooting user identity issues on firepower systems can be quite daunting at times if you do not know where exactly you should look for issues. After encountering many different problems I thought it might be a good idea write down what I have learned from various troubleshooting sessions since unfortunately the amount of available documentation is quite lacking.

<!--more-->

## Architecture

Before diving into the various tools we can use to troubleshoot identity issues lets focus on how exactly passive user identity is working and how we are able to accurately map users to ip addresses.

In most scenarios we are using a LDAP directory to fetch user to ip mappings. Because users are using a central point to authenticate to the corporate network we can query our directory service (e.g. Active Directory) to find out which ip address generated a logon event. To fetch this information we can use two different approaches.

The first approach is using a dedicated agent that must be installed on a domain-joined windows server. The Firepower User Agent (FUA) will be used to query the active directory security log for logon events and send this information to FMC using a REST interface.

The second approach is using Cisco ISE integration. ISE can natively join a active directory domain to query the security log for logon events. This information is then propagated to FMC using pxGRID. pxGRID is basically just a message bus that is used by FMC to subscribe to identity information of ISE.

Apart from fetching ip mappings using either FUA or ISE, Firepower Management Center will also directly query the ldap directory to download user to group mappings. A so called "Realm" is configured on FMC to periodically download this information to FMC to keep track of any changes that might occur (e.g. new user added to group HR, users being deleted, etc.)

The last step in the chain is propagating the received user to group mappings and user to ip mappings to the sensors. To ensure the identity information is sent to the correct sensors, we must associate a so called identity policy to our access-control-policy in FMC. The identity policy will define which realm to use and for which network ranges we want to send identity mappings to our sensors (e.g. send ip to user mappings to sensor if ip address is within RFC1918 ranges).

After the access-control-policy with the associated identity policy has been successfully deployed to our sensor, FMC will  forward identity information to the sensor using an tls encrypted tunnel that is used for communication between FMC and the sensor. The so-called sftunnel (Sourcefire Tunnel) component is used for communication between the two devices and is utilized for various purposes (monitoring, configuraton deployment, updates, etc.). A snapshot of the currently available mappings is pushed by FMC to the sensor over the sftunnel, which is then processed by the SFDataCorrelator process.

### TL;DR

1. Identity integration is implemented centrally using FMC
2. FMC connects directly to the directory service to download user to group mappings (realm configuration)
3. FMC connects to either Firepower User Agent or ISE pxGRID to receive user to ip mappings
4. An identity policy must be assigned to the access-control-policy to send identity information to sensors
5. FMC pushes identity information to sensors using a tls encrypted tunnel named sftunnel

## Basic Troubleshooting

This section contains various troubleshooting steps that can be performed to solve common identity related issues.

### Verify Connectivity between FMC and ISE

Under the Identity Sources menu in the FMC UI, the connection status between FMC and ISE can be tested. Since pxGRID will only be active on the Primary Administration Unit (PAN) on ISE, you will see the connection to one ISE failing. This is perfectly fine.

Navigate to **System > Integration > Identity Sources > Identity Service Engine** to verify the connection status

![pxgrid02]({{ site.url }}/images/posts/2017-01-14-Troubleshooting-Firepower-Identity/troubleshooting_ise_pxgrid_02.png)

You may use the **Additional Logs** section to check for error information. Normally issues can be traced back to connectivity issues between FMC and ISE (e.g. missing firewall rules) or certificate mismatches.

Another method of checking the pxGRID connection is to directly connect to ISE and check the pxGRID Services status. If FMC is listed as Subscriber for identity information you are good to go.

To check the status of pxGRID on ISE navigate to **Administration > pxGrid Services > Clients** on the PAN.

![pxgrid01]({{ site.url }}/images/posts/2017-01-14-Troubleshooting-Firepower-Identity/troubleshooting_ise_pxgrid_01.png)

#### Additional Troubleshooting information

If you are looking for additional troubleshooting logs for pxGRID you may want to check out /var/log/messages on FMC. Use the following command to check for ISE related log entries:

```
cat /var/log/messages | grep adi.ISEConnection
```

### Verify Connectivity between FMC and Active Directory

You may use the test function in the realm configuration to check the connection to a directory server. In FMC navigate to **System > Integratrion > Directory** and edit the configured realm. A test button will appear which can be used to connect to the server.

![fmc01]({{ site.url }}/images/posts/2017-01-14-Troubleshooting-Firepower-Identity/fmc_realm_troubleshooting_01.png)

#### Additional Troubleshooting information

If you are looking for additional troubleshooting logs for directory connections, you may want to check out /var/log/messages on FMC. Use the following command to check for directory related events that will indicate connection failures or successful directory connections.

```
cat /var/log/messages | grep adi.Directory
```

### Verify Access Control Policy Configuration

To make sure that identity mappings are pushed from FMC to your sensor make sure that your identity policy is referenced in your access-control policy.

## Advanced Troubleshooting

This section contains troubleshooting information for advanced issues. Use the following procedures at your own risk.

### Debugging ADI

Firepower Management Center and the sensor use the ADI process for identity related tasks. For example on FMC, ADI is used to connect to ldap directories, establish a connection to your FUA or ISE. To gather additional debug information you may start the process using a debug flag. Keep in mind that you have to kill the process before that, which will stop identity updates to your FMC. If you depend on identity integration dont leave it disabled for too long...

1. Login to FMC using SSH and change to root

   ```
   sudo su -
   ```

2. Disable adi process using pmtool

   ```
   pmtool DisableById adi
   ```

3. Verify adi is not running anymore

   ```
   ps ax | grep adi
   ```

4. Start adi process with debug flag and pipe output to `/var/tmp/adi-debug.log`

   ```
   nohup /usr/local/sf/bin/adi --debug > /var/tmp/adi-debug.log 2>&1 &
   ```

5. Verify adi is running again

   ```
   ps ax | grep adi
   ```

6. After gathering debug information kill adi again (use ps ax to find out pid)

   ```
   kill -9 <adi-pid>
   ```

7. Enable adi process using pmtool

   ```
   pmtool EnableById adi
   ```

### Dump Identity Mappings

Firepower Management Center saves identity mappings to its Mysql database. This information is then sent to the sensor using sftunnel. A file containing the current mappings is transferred to the sensor and processed by SFDataCorrelator which will write the current identity mappings to the local mysql database on the sensor.

If everything is working corretly you should find the user to ip mapping in three different locations

1. Mysql database on FMC
2. Identity snapshot on sensor
3. Mysql database on sensor

In case a user to ip mapping is not present anywhere down the road you probably hit a bug and should contact TAC.

#### FMC Mysql database

1. Connect to FMC using SSH and login to mysql database. Use the default password 'admin'.

   ```
   mysql -uroot -p sfsnort
   ```

2. Search database for the user that experiences issues (note user id after executing query)

   ```
   mysql> select * from user_identities where username like '%<your-username>%' \G
   ```

3. Search database for user to ip mapping for the specified user (use user id for query)

   ```
   mysql> select inet6_ntoa(ipaddr) FROM current_user_ip_map where id=<user-id> \G
   ```

#### Sensor Identity Snapshot

The user to ip snapshot is not human readable and must be dumped using the `u2dump` utility. Before dumping the identity mappings verify that a snapshot exists.

1. Connect to sensor using SSH, change to expert mode and change to root

   ```
   > expert
   sudo su -
   ```

2. Verify existance of user snapshot

   ```
   ls -lah /var/sf/user_enforcement
   ```

3. Dump user snapshot to /var/tmp

   ```
   u2dump /var/sf/user_enforcement/user_ip_map.* > /var/tmp/user-ip-map.dump
   ```

4. Open user snapshot using `vi`(you might also use another text editor or use `more` but I prefer vi to search)

   ```
   vi /var/tmp/user-ip-map.dump
   ```
