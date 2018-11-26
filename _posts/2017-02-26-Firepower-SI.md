---
layout: post
title:  "Firepower Security Intelligence"
date:   2017-04-15
tags: [firepower, security-intelligence]
---

Security Intelligence (SI) is used to block connections based on reputation on firepower systems. It can be used to block traffic based on IP, DNS and URL and is useful as a first line of defense to blacklist undesirable connections from/to public ip addresses.

<!--more-->

Security Intelligence is very easy to configure and is applied globally to an access-control-policy. You can either use cisco provided feeds or upload your own SI feeds that can be polled frequently. (e.g. via http from a webserver). Another option would be manually creating a list which can be edited in the object manager, which is rather static. If possible try to use a feed that is automatically downloaded for more flexibility. 

In case you encounter any false positives and legitimate traffic is being blocked by SI you can whitelist SI entries using the context explorer or the object manager. 

If you want to use custom feeds you can create your own feed or use publicly available feeds like these:

* Cisco Talos: http://www.talosintelligence.com/feeds/ip-filter.blf
* Malc0de: http://malc0de.com/bl/IP_Blacklist.txt
* SANS: https://isc.sans.edu/feeds/suspiciousdomains_High.txt

The cisco provided feeds are categorized to allow granular control on which elements should be blocked. As a general rule of thumb I would not use the spam feed for my installations since I don't want my firewall to block mail traffic, which should be filtered by a email security appliance which provides more granular control on what to do with undesired content.

When using SI keep in mind that some changes to the feeds will require you to manually re-deploy your configuration for changes to take effect. See the table below for more information: 

| Object Type           | Edit capabilities                        | Redeploy required after edit?            |
| --------------------- | ---------------------------------------- | ---------------------------------------- |
| Default Lists         | Add entries using context menu, delete entries using object manager | No, after adding entries. Yes after deleting entries |
| Custom Lists          | Upload new/replacement lists using object manager | Yes                                      |
| System-provided feeds | Modify update frequency                  | No                                       |
| Custom feeds          | Modify using object manager              | No                                       |
| Sinkhole              | Modify using object manager              | Yes                                      |

## Configuration

SI configuration consists of two (or three for DNS SI) essential steps:

1. Configure your feeds & polling intervals
2. Configure a DNS policy for DNS feeds (not covered in this post)
3. Apply SI to you access control policy

To configure a new feed navigate to **Objects > Object Management > Security Intelligence** in FMC UI and choose the appropriate feed type to configure. In this example we will add the malc0de blacklist to FMC and set the update frequency to 30 minutes to make sure our feed is always up to date. 

![si01]({{ site.url }}/images/posts/2017-04-15-Security-Intelligence/si-01.png)

After pressing ok, the feed will be downloaded to FMC and you may proceed with configuring your access control policy to use your SI feed. Navigate to **Policies > Access Control**, select your ACP and navigate to the **Security Intelligence** tab.

Search for the newly defined SI feed and add it to the blacklist. After you are done with the configuration you have to deploy your configuration for changes to take effect.

![si01]({{ site.url }}/images/posts/2017-04-15-Security-Intelligence/si-02.png)

## Traffic Flow

Security Intelligence is being enforced within SNORT. If you want to whitelist traffic that would be blocked by SI you will need to create a rule in your pre-filter policy (if you use FTD) or add the false positive to your whitelist to make sure traffic to/from the address is not being blocked.

![si03]({{ site.url }}/images/posts/2017-04-15-Security-Intelligence/si-03.png)

## Troubleshooting

If you are interested in in the content of the downloaded feeds you will need to SSH into FMC and check the feeds via CLI. Unfortunately there is no way to use the FMC UI to check the content of the downloaded feeds yet.

FMC downloads the feeds to the following locations

| Type | Path                   |
| ---- | ---------------------- |
| IP   | /var/sf/iprep_download |
| DNS  | /var/sf/sidns_download |
| URL  | /var/sf/siurl_download |

If you want to check the feeds for a specific entry you can use grep to filter the content of the feeds. Using the -H flag you can display the ID of the feed where the entry was found,

```
admin@firesight:/var/sf/iprep_download$ grep -H 31.210.117.131 *-*
032ba433-c295-11e4-a919-d4ae5275a468:31.210.117.131
```

### Feed downloads

If you would like to verify that your feed download is working correctly you can use the **Download Feeds** option in FMC UI. Unfortunately the progress of the download is not being displayed in the task view of FMC that's why you will need to check the /var/log/messages logfile to see if the download has been successful.

![si04]({{ site.url }}/images/posts/2017-04-15-Security-Intelligence/si-04.png)

```
Apr 15 16:17:19 firesight SF-IMS[3911]: [3962] CloudAgent:IPReputation [INFO] List 3d8b6b1e-21f3-11e7-a642-c0759f5522e3 being updated up_freq: 0 need_update: 0 
Apr 15 16:17:19 firesight SF-IMS[3911]: [3962] CloudAgent:IPReputation [INFO] Response code: 200 
Apr 15 16:17:19 firesight SF-IMS[3911]: [3962] CloudAgent:IPReputation [INFO] Content type: text/plain
Apr 15 16:17:19 firesight SF-IMS[3911]: [3962] CloudAgent:IPReputation [INFO] Download size in bytes: 729 
Apr 15 16:17:19 firesight SF-IMS[3911]: [3962] CloudAgent:IPReputation [INFO] Download speed bytes/sec: 3192
Apr 15 16:17:19 firesight SF-IMS[3911]: [3962] CloudAgent:IPValidator [INFO] Processing file /var/sf/iprep_download/tmp/3d8b6b1e-21f3-11e7-a642-c0759f5522e3
Apr 15 16:17:19 firesight SF-IMS[3911]: [3962] CloudAgent:IPReputation [INFO] Successfully downloaded 3d8b6b1e-21f3-11e7-a642-c0759f5522e3 <--- Download has been successful!

```

If you have multiple feeds you will want to check the /etc/sf/iprep_sources.conf configuration file to verify the ID of your feed. 

```
cat /etc/sf/iprep_sources.conf
(...)
    3d8b6b1e-21f3-11e7-a642-c0759f5522e3 <--- ID of the Feed
    {
        ITEM_NAME malcode; <--- Name in FMC UI
        HOST_URL http://malc0de.com/bl/IP_Blacklist.txt; 
        UPDATE_FREQ 6; <-- Frequency in 5min steps (5 * 6 = 30 minutes configured)
        NEED_UPDATE 1;
    }
```

As you can see the ID referenced in /var/log/messages matches our malc0de blacklist. Now that you know the ID of the feed you can also check the downloaded file of our ip reputation feed which is being saved to /var/sf/iprep_download/<feed_id>

```
admin@firesight:~$ more /var/sf/iprep_download/3d8b6b1e-21f3-11e7-a642-c0759f5522e3
```

```
#malcode
104.214.150.216
54.230.87.253
162.249.2.136
183.47.234.108
85.159.66.172
121.41.10.159
52.84.125.52
183.47.234.107
(...)
```

