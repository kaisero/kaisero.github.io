---
layout: post
title:  "Firepower Process Management"
date:   2017-01-26
tags: [firepower, pmtool, troubleshooting]
---

Have you ever been in a situation where you wanted to troubleshoot a certain firepower process? Tried to kill a process, to re-start it with some debug flags but it was already re-started automatically by process manager? Well pmtool is here to help. Using pmtool you can disable services, restart processes and check dependencies between components on firepower systems.

<!--more-->


## Disclaimer

As always you should be aware of what you are doing. With great power comes great responsibility, so do not disable processes and then complain that a feature is not working anymore. Before using this utility on production systems, make sure you understand possible ceveats that could result from disabling and restarting processes on the firepower system.

## pmtool

### Introduction

pmtool is a binary to manage processes on firepower systems. It can be used from an FMC root shell and directly in the sfcli on firepower sensors like FTD, ASA with Firepower Services or the dedicated NGIPS.

Make sure to run pmtool as root. 

```
admin@fmc:~$ sudo su -
```

### Process Status

If you want to check the status of processes you can use the `Status` option for `pmtool`. Because the command will list the status for all managed daemons and tasks I have shortened the output and uploaded the full output [here]({{site.url}}/output/posts/2017-01-26-Firepower-Process-Management/pmtool-status.txt).

```
root@fmc:~$ pmtool status
```

```
Received status (0): 

Global Environment:
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin

Daemons:
SFDataCorrelator (normal) - Running 6703
Command: /usr/local/sf/bin/SFDataCorrelator --nodaemon
PID File: /var/sf/run/SFDataCorrelator.pid
Enable File: /etc/sf/SFDataCorrelator.run
CPU Affinity: 
Priority: 0
Stop Timeout: 300
Next start: Thu Jan 26 12:18:16 2017
Requires: mysqld
CGroups: memory=System(enrolled)

(...)
```

In case you only want to check a specific process you may grep for the name. In the following example we will check the status of the identity daemon named adi.

```
root@fmc:~# pmtool status | grep adi
adi (normal) - Running 6209
Command: /usr/local/sf/bin/adi
PID File: /var/sf/run/adi.pid
```

### Disable/Enable  Processes

In case you want to disable a process for troubleshooting reasons (e.g. start daemon manually and redirect debug output to a logfile) you can use the `DisableById` option for `pmtool`. Keep in mind that you will have to enable the process again afterwards or it wont be under ProcessManagers control again.

Before disabling a process, make sure it is actually running. If it is not running there must be some issue. 

Check status of identity process

```
root@fmc:~# pmtool status | grep adi
adi (normal) - Running 6209
Command: /usr/local/sf/bin/adi
PID File: /var/sf/run/adi.pid
```

Disable identity process

```
root@fmc:~# pmtool DisableByID adi
```

By issueing the status command once again you can verify that the process has indeed been disabled

```
root@fmc:~# pmtool status | grep adi
adi (normal) - User Disabled
Command: /usr/local/sf/bin/adi
PID File: /var/sf/run/adi.pid
```

After you are done troubleshooting dont forget to enable the process again and make sure it is indeed running again

```
root@fmc:~# pmtool EnableByID adi
```

```
root@fmc:~# pmtool status | grep adi
adi (normal) - Running 2705
Command: /usr/local/sf/bin/adi
PID File: /var/sf/run/adi.pid
```

### Restart Process

If you get into a situation where a process is misbehaving (e.g. SFDataCorrelator not loading URL database into snort engine, adi not connecting to pxGRID correctly, snort dropping traffic unexpected, etc.) you may restart a process. Make sure you are aware of the function of the process and how it should behave if you restart it.

For our example we will restart the ips engine processes (snort) on one of our sensors because we experience very high cpu usage that can be tracked back to snort and traffic is being dropped unexected. Be aware that restarting snort may lead to temporary traffic loss for up to a few seconds.

if you want to restart all snort processes make sure you use the restartByType keyword and not restartById

```
root@ftd:/home/admin# pmtool restartByType snort
```



