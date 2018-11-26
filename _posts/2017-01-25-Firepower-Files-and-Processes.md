---
layout: post
title:  "Firepower Files and Processes"
date:   2017-01-25
tags: [firepower, troubleshooting]
---

Since I had to use the root shell various times for troubleshooting  on firepower systems, I  decided to document some of the various binaries and logfiles that are available on FMC and firepower sensors.

<!--more-->

The following list only containts an overview of the various tools you can find on fmc and ftd shell. In the future I will create blog posts for various items listed here to explain what they are doing and how to use them.


## Disclaimer

This list of configuration files, binaries, processes and log files has been created for anyone who wants to take a deeper look into the system. Keep in mind that this list is not a complete reference and only consists of elements I found useful. Before touching any binaries and processes in production environments make sure you really know what you are doing. Descriptions for various files may not be entirely correct since many of the listed tools are not documented by Cisco in any way for customers and partners. If you spot any errors just let me know. 

## FirePOWER Management Center

### Processes & Binaries

| Path                                   | Description                              |
| -------------------------------------- | ---------------------------------------- |
| /usr/local/sf/bin/adi                  | Identity Process (Active Directory/pxGRID/User Agent) |
| /usr/local/sf/bin/syncd.pl             | HA Daemon for FMC High Availability      |
| /usr/local/sf/bin/CloudAgent           | Cloud Agent (AMP, URL Filtering, SI)     |
| /usr/local/sf/bin/sftunnel             | Management SSL Tunnel                    |
| /usr/local/sf/bin/sftunnel_status.pl   | Check sftunnel status                    |
| /usr/local/sf/bin/pmtool               | FMC Management Binary (Control Processes, Display Process Health, etc.) |
| /usr/local/sf/bin/stats_unified.pl     | Check sftunnel event transfer status     |
| /usr/local/sf/bin/manage_estreamer.pl  | Manage eStreamer                         |
| /usr/local/sf/bin/manage_pruning.pl    | Manage pruning (e.g. clear event db)     |
| /usr/local/sf/bin/manage_HADC.pl       | Manage FMC High Availability             |
| /usr/local/sf/bin/troubleshoot_HADC.pl | Troubleshoot FMC High Availability       |
| /usr/local/sf/bin/OmniQuery.pl         | Connect to Sybase Database               |
| /usr/local/sf/bin/ids_event_db_info.pl | Check IDS event rate of the last hour    |
| /usr/local/sf/bin/eo_tool              | Object Management Tool of FMC application. Do not edit objects if you do not know what you are doing |
| /usr/local/sf/bin/pigtail              | Tail various logfiles for troubleshooting |
| /usr/local/sf/bin/u2dump               | Dump user identity mappings into a human readable format |

### Configuration Files

| Path                          | Description                          |
| ----------------------------- | ------------------------------------ |
| /etc/sf/PM.conf               | Process Manager configuration        |
| /etc/sf/ADI.conf              | Identity Process configuration       |
| /etc/sf/sftunnel.conf         | SSL Tunnel configuration             |
| /etc/sf/fireAMP_proxy.conf    | AMP Proxy Settings                   |
| /etc/sf/ims.conf              | Environment Variables                |
| /etc/sf/ims-data.conf         | Snort Authentication Credentials     |
| /etc/sf/bca.cfg               | Brightcloud URL Filtering            |
| /etc/sf/cloudagent.conf       | Cloud Agent (AMP, URL Filtering, SI) |
| /etc/sf/iprep_sources.conf    | Security Intelligence IP Feeds       |
| /etc/sf/dns_sources.conf      | Security Intelligence DNS Feeds      |
| /etc/sf/dns_cache.conf        | DNS Caching Options                  |
| /etc/sf/network-amp.conf      | AMP for Network Settings             |
| /etc/sf/amp-stunnel.conf      | AMP Cloud Settings                   |
| /etc/sf/sandbox_cloud.conf    | Threatgrid Cloud Settings            |
| /etc/sf/sandbox_file_size.cfg | Threatgrid max Filesize              |
| /etc/sf/geo_updates.conf      | Geo-IP Update Settings               |
| /etc/sf/seu_versions.conf     | Snort Version                        |
| /etc/sf/email.conf            | Mail settings                        |
| /etc/sf/msmtprc               | Mail setting details                 |
| /etc/sf/patch_history         | Patch History                        |
| /etc/sf/sf-version            | OS / APP Version                     |
| /usr/local/sf/updates/        | Update Directory                     |

### Log Files

| Path                                     | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| /var/log/messages                        | Logging for various proccesses           |
| /usr/local/sf/cloud_download/tmp/url_db_dl.log | Brightcloud Database Download Log        |
| /var/log/urldb_log                       | Brightcloud Database Download Log        |
| /var/log/iprep.log                       | Security Intelligence Feed Download Status Log |
| /var/log/smart_agent                     | Smart Licensing Agent Log                |
| /var/log/sch.log                         | Call Home Log                            |
| /var/log/ntp.log                         | NTP Server Connections                   |
| /var/log/process_stdout.log              | STDOUT Output of Processes               |
| /var/log/process_stderr.log              | STDERR Output of Processes               |
| /var/log/CSMAgent.log                    | CSM related access logs                  |
| /var/log/mojo.log                        | Mojo Perl Webserver Logs                 |
| /var/log/syncd.log                       | High Availability Log (FMC HA)           |
| /var/log/sf/<version>/status.log         | Status Log for FMC upgrade               |
| /var/log/sf/<version>/000_start/*        | Logs for actions taken before upgrade is started |
| /var/log/sf/<version>/200_pre/*          | Logs for actions taken to start update   |
| /var/log/sf/<version>/300_os/*           | Update logs for Fire Linux OS upgrade    |



## FirePOWER Threat Defense

### Processes & Binaries

| Path                                     | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| /ngfw/usr/local/sf/bin/pmtool            | FirePOWER Management Binary (Control Processes, Display Process Health, etc.) |
| /ngfw/usr/local/sf/bin/CloudAgent        | Cloud Agent (AMP, URL Filtering, SI)     |
| /ngfw/var/cisco/ngfwWebUi/tomcat/bin/ngfw_onbox_start_tomcat.sh | Onboard Web UI (FDM)                     |
| /ngfw/usr/local/sf/bin/sftunnel          | Management SSL Tunnel                    |
| /ngfw/usr/local/sf/bin/sf_troubleshoot.pl | Generate troubleshooting file for sensor. Saved to /ngfw/var/common |

### Configuration Files

| Path                       | Description            |
| -------------------------- | ---------------------- |
| /etc/sf/bca.conf           | URL Filtering Settings |
| /etc/sf/sandbox_cloud.conf | ThreatGRID Settings    |
| /etc/sf/cloudagent.conf    | AMP and SI Settings    |
| /etc/sf/patch_history      | Patch History          |


### Log Files

| Path                                | Description                    |
| ----------------------------------- | ------------------------------ |
| /ngfw/var/log/process_stderr.log    | STDERR Output of FTD Processes |
| /ngfw/var/log/process_stdout.log    | STDOUT Output of FTD Processes |
| /ngfw/var/log/ngfwManager.log       | ngfwManager Log                |
| /ngfw/var/log/messages              | General Log File               |
| /ngfw/var/log/action_queue.log      | Task Log (FMC Triggered Tasks) |
| /ngfw/var/log/policy_deployment.log | Policy Deployment Log          |

### Other Files

| Path                                     | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| /var/sf/sidns_download                   | Security Intelligence - DNS              |
| /var/sf/sidns_download/*.lf              | DNS Feeds                                |
| /var/sf/siurl_download                   | Security Intelligence - URLs             |
| /var/sf/siurl_download/*.lf              | URL Feeds                                |
| /var/sf/iprep_download                   | Security Intelligence - IPs              |
| /var/sf/iprep_download/.*lf              | IP Reputation Feeds                      |
| /var/sf/cloud_download                   | Brightcloud URL Filtering                |
| /var/sf/cloud_download/cloudagent_dlupdate_health | Brightcloud URL Filtering database status |
| /var/sf/cloud_download/full_bcdb_rep.bin | URL Database                             |
| /dev/shm/Global.bcdb                     | URL Database (in shared memory)          |
| /var/sf/clamupd_download/                | CLAMAV Database                          |
| /var/sf/clamupd_download/*.cvd           | CLAMAV Database files                    |
| /var/sf/remediation                      | Remediation modules                      |
| /var/sf/detection_engines                | Snort configuration                      |
| /var/sf/updates                          | Updates directory                        |
| /var/sf/identity_integration             | User to IP mappings                      |
| /var/common                              | Dump Directory (backups, t-shoot files, etc.) |
