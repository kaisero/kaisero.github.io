---
layout: post
title:  "Troubleshooting Firepower URL Filtering"
date:   2017-01-24
tags: [firepower, url filtering, troubleshooting]
---

URL Filtering is a useful feature to block malicious domains or block unwanted web traffic. It is a feature that is easy to configure but has some hidden caveats. In this post we will look at some limitations and ways to troubleshooting url filtering related issues on firepower systems.

<!--more-->

## Architecture

Lets start off by a quick introduction on how exactly URL Filtering works and what happens behind the scenes on a firepower system. URL Filtering is a L7 feature that will let you block web traffic based on URLs, Categories and Reputation. There are basically two different ways to use url filtering. First is creating your own lists and reference them in your access control policy. The second method is using the built-in url database for url categorization and  reputation scores. The database in question is Brightcloud which is provided by Webroot. It is commonly used in security products like Palo Alto Networks NGFW and also in Cisco Firepower for URL Intelligence.

The brightcloud database is pushed from FMC to sensors with the url filtering license. To keep the url filtering database up to date it is recommended to enable automatic updates on FMC. By using this feature FMC will connect to brightcloud every 30 minutes to download any available updates and distribute the update to the sensors. To make sure you have automatic updates enabled navigate to **System > Integration > Cisco CSI**

![url01]({{ site.url }}/images/posts/2017-01-24-Troubleshooting-Firepower-URL-Filtering/url-filtering-01.png)

In case an URL cannot be found in the local database the option **Query Cisco CSI for Unknown URLs** can be enabled on FMC to lookup unknown URLs on demand.

### Download Process

As described above the url filtering database is first downloaded to FMC. This is done by the CloudAgent process which will save the database to `/var/sf/cloud_download/` on the FMC filesystem. The database is then pushed to sensors using sftunnel. sftunnel is the process which is used for communication between FMC and sensors. Data and files are being exchanged using the tls encrypted tunnel.

After uploading the file to the sensor filesystem at `/var/sf/cloud_download/` the SFDataCorrelator process will move the database into the shared memory space at `/dev/shm` for performance reasons. From there the snort process (ips engine) will access the new database file for url lookups.

## Limitations

URL Filtering is subject to some limitations as of version 6.1.0.1. This may change in the future. 
In case any of the listed limitations are removed,  I will update this post.

### URL Shortening

Categorization for URLs that are obfuscated by using a url shortening service is not working. An enhancement request has been opened to address this issue. (CSCvc54061)

### Response Pages

When a response page is used to inform the user of blocked content, the url category is not displayed. This may lead to confusion for users since they do not know why a site has been blocked. An enhancement request has been opened to address this issue. (CSCvc85039, CSCuv11331)

### Subdomain Categorization not working correctly

Subdomain categorization of URLs does not work on certain sensors. Because of resource limitations on low to mid-end ASA devices,  a smaller local database is used which does not include subdomains. This results in subdomains being categorized the same as the parent domain even though brightcloud might categorize the subdomain differently (e.g. mail.google.com is not a search engine). This behavior can be observed on ASA 5506-X up to ASA 5525-X.

In my opinion this is a bug, if you want to know more check out this Bug ID: CSCuz66673, CSCuy49952

### Selective URL-based block

Blocking content within a website but allowing everything else is not possible. If you want to block embedded youtube videos on a website you will not be able to do so. An enhancement request has been opened to address this issue. (CSCvc78257)

### Blocking Unknown URLs

If an URL cannot be found in the local database and cloud lookup fails the category for the URL will remain "Unknown". In this case no additional action can be defined to permit/deny traffic.  An enhancement request has been opened to address this issue. (CSCuw40785)

## Basic Troubleshooting

This section contains various troubleshooting steps that can be performed to solve common identity related issues.

### Verify URL Categorization

The simplest way to verify if an URL is correctly categorized is by checking the connection events on FMC. Navigate to ** Analysis > Connections > Events** and check the URL Category column.

![url02]({{ site.url }}/images/posts/2017-01-24-Troubleshooting-Firepower-URL-Filtering/url-filtering-02.png)

### Verify last URL Database update

In case you  encounter issues with incorrect categorization you might wanna check when  your url database has been updated the last time.

To check the last update  navigate to **System > Integration > Cisco CSI**

![url03]({{ site.url }}/images/posts/2017-01-24-Troubleshooting-Firepower-URL-Filtering/url-filtering-03.png)

If you are looking for previous update results you must login to fmc using ssh. Results of url database downloads are saved to `/var/log/urldb_log`. Keep in mind that the date is displayed in unix timestamp format so you will have to convert it to a human readable format.

To display previous url database downloads execute the following command
```
cat /var/log/urldb_log
```

In case you want to check the date of timestamp use the following command on a timestamp

```
date -d @1485216692
```

A successful download looks like this.

```
1485216692,0,Successfully downloaded,
applied and moved,full_bcdb_rep_5.79.bin,2f8d7a54712823e6d81f1844d13e50c5
```

If there has been no update you will receive this message. Since you already have the current database this log entry is perfectly fine

```
1485040265,0,Up to date,
```

This message indicates an error during the url database download.

```
1484955667,1,Download Failed,
```

Another way to verify that the database has been downloaded successfully is by checking `/var/log/messages`. You should see look entries like this if the download was successful

```
cat /var/log/messages | grep -E "CloudAgent.*bcdb"
```
```
CloudAgent:CloudAgent [INFO] **new file name is full_bcdb_rep_X.X.bin
CloudAgent:CloudAgent [INFO] checksum of file full_bcdb_rep_X.X.bin is c6eefedaf682cb4274e6599978ebe662
CloudAgent:CloudAgent [INFO] The original file is /var/sf/cloud_download/tmp/part_bcdb_rep_X.X.bin
CloudAgent:CloudAgent [INFO] The renamed file is /var/sf/cloud_download/part_bcdb_rep_X.X.bin
CloudAgent:CloudAgent [INFO] The 20M db not to be removed is full_bcdb_rep_X.X.bin
```

## Advanced Troubleshooting

### All URLs categorized as unknown

If all URLs are categorized as unknown and the url database has been downloaded successfully to FMC you probably hit an issue on your sensor. From what I have seen so far this can happen if the SFDataCorrelator process  does not load a new database from the shared memory segment into snort correctly.

First you want to check if the database has been pushed to the sensor correctly. The file transfer from FMC to the sensor is logged to `/var/log/messages`

```
cat /var/log/messages | grep -E "sftunnel.*bcdb"
```

You should see log entries similar to the following that indicate that the database has been pushed correctly

```
sftunneld:stream_file [INFO] stream_src=/var/sf/cloud_download/full_bcdb_rep_X.X.bin
sftunneld:stream_file [INFO] stream_dest=/var/sf/cloud_download/tmp/full_bcdb_rep_X.X.bin
sftunneld:stream_file [INFO] FILE /var/sf/cloud_download/full_bcdb_rep_X.X.bin
sftunneld:stream_file [INFO] Initiated fstream task pushing /var/sf/cloud_download/full_bcdb_rep_X.X.bin to fif2101-7b4f-11e6-b4a1-ca59e03bf0e4, task_id 449
sftunneld:stream_file [INFO] Opened file '/var/sf/cloud_download/full_bcdb_rep_X.X.bin'
```

On the sensor side you can check `/var/log/messages` to verify that the database has been received and is being written to the shared memory segment. On FTD you will have to check `/ngfw/var/log/messages` on the bash shell (expert mode).

To access bash use the expert command  on FTD and change user to root on bash shell

```
> expert
admin@ftd:/home/admin# sudo su -
```

```
root@ftd:/home/admin# cat /ngfw/var/log/messages | grep -i url
```

```
SFDataCorrelator:HandleUserMessage [INFO] CHANGE_USER_RELOAD_URLDB message
SFDataCorrelator:URLUserIP_CorrelatorThread [INFO] URLDB file /ngfw/var/sf/cloud_download/full_bcdb_rep_x.x.bin is xxx MB
SFDataCorrelator:URLUserIP_CorrelatorThread [INFO] Memory available prior to URLDB Load is xxxx MB
SFDataCorrelator:URLDBLookup [INFO] Found 1 files:
SFDataCorrelator:URLDBLookup [INFO] Files are full_bcdb_rep_x.x.bin
SFDataCorrelator:URLDBLookup [INFO] Updated BcSDK found with SlotInfo function
SFDataCorrelator:URLDBLookup [INFO] CurrentDb is 1, nverion1 is XXXXX, nversion2 is XXXXX and nActiveInstances is X
SFDataCorrelator:URLDBLookup [INFO] Loading the URLDB File full_bcdb_rep_5.9.bin
SFDataCorrelator:URLDBLookup [INFO] Updating Current Database data, full_bcdb_rep_X.X.bin X.X
SFDataCorrelator:URLUserIP_CorrelatorThread [INFO] Loaded URL DB into shared memory
sfpreproc:URLDBLookup [INFO] Success, attached to database
sfpreproc:DataMessaging_UserGroupUrlAPI [INFO] Swapped shmem db pointers
```

The important log entry is `Swapped shmem db pointers` at the end of the log which indicates that the new database is now in use. If you do not see this log entry you probably hit a bug.

A possible quick fix is rebooting the sensor or restarting the SFDataCorrelator process.

To restart the SFDataCorrelator process you can use pmtool (restarting SFDataCorrelator should not have any impact on traffic forwarding.) 

```
root@ftd:/home/admin# pmtool restartById SFDataCorrelator
```
