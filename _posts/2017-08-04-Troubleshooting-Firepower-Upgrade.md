---
layout: post
title:  "Troubleshooting Firepower Upgrade"
date:   2017-08-04
tags: [firepower, sensor, upgrade, troubleshooting, ftd, firepower-services, fmc]
---

At some point we have all come across update issues with error messages like "Update install failed.", without any further details available. In my opinion there should be more details on an UI to further troubleshoot issues like that, but when it comes to upgrade procedures on FMC that is about it. 

So how exactly should we start analyzing upgrade issues on FMC? Although the UI output is rather generic there is lots of information to be found using the CLI. Each upgrade procedure consists of a variaty of scripts that are being executed on the device that is being upgraded.

<!--more-->

In my example I have started an upgrade of a firepower module which failed with the following message:

![upgrade-failure]({{ site.url }}/images/posts/2017-08-04-Troubleshooting-Firepower-Upgrade/deployment-failure.png)

To look into the logs you will have to ssh to our sensor and change to expert mode (bash)

```
> expert
```

Depending on your platform you will find the upgrade logs at the following locations:

```
/var/log/update.status (Firepower module)
/ngfw/var/log/update.status (FTD)
```

You can go ahead and use cat/less/more/vi etc. to open the file. At the end of the file an error message should be logged indicating why exactly the installation failed.

In my case I received the following error message:

```
OUT: **********************************************************
OUT: [170706 14:47] Starting script: 200_pre/006_check_snort.sh
OUT: Entering 200_pre/006_check_snort.sh...
OUT: MODEL is Elektra sensor
OUT: Snort version is too old. Please apply AC Policy from DC before attempting upgrade.
OUT:
OUT: [170706 14:47:46] Fatal error: Error running script 200_pre/006_check_snort.sh
OUT: [170706 14:47:46] Exiting.
```

Due to the verbosity of the output it was easy to spot the issue right away. I had upgraded my FMC from 6.1.0.1 to 6.2.0.2 without re-applying my policy to my sensors. This caused an issue during my sensor upgrade that could be resolved by deploying my configuration again and restarting the upgrade procedure.

## Is that all?

In case you are looking for more detailed troubleshooting data you can go ahead and checkout the logs of the different scripts that run during the upgrade procedure. In the `/var/log/sf` or `/ngfw/var/log/sf/` directory you will find a directory for the upgrade you have attempted named `Cisco_Network_Sensor_Patch-<VERSION>`. You can go ahead and analyze the output of the different scripts to find more information on your specific issue.





