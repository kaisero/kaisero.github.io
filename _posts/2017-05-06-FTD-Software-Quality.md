---
layout: post
title:  "FTD Software Quality"
date:   2017-05-05
tags: [firepower, ftd, qa, testing]
---

It has been over a year since the release of Firepower Threat Defense and due to some recent announcements I thought it would be a good time to take a look at current challenges we face with FTD and how Cisco is trying to get back on the track.

<!--more-->

## Feature Parity

One of the major reasons FTD has not been adopted (yet) is due to feature parity with ASA. In my opinion Cisco should have waited up until 2017 to release FTD since it was released with so many missing features that users felt like using a beta release that missed very important pieces that ASA was offering.This created some very ambitious roadmaps for FTD.

So what exactly was missing in the first FTD release?

* Site-to-Site VPN
* Client-to-Site VPN (AnyConnect, SSLVPN)
* Multiple Context Mode
* Multicast Routing
* Policy-based Routing
* EIGRP, IS-IS Routing
* BFD
* Netflow Export
* VxLAN
* Modular Policy Framework
* Inter-Chassis Clustering
* On-box management
* REST API
* ASA Migration Utility

This list is by far not complete but a very good overview of many important features that were missing at the initial release. So as of today where do we stand? Cisco was able to deliver some of the listed items but unfortunately there is still much to be done when it comes to feature parity and stability.

* Site-to-Site VPN - delivered with 6.1.0 (only between FTD devices), 6.2.0 introduced support to other vendors
* Multicast Routing: delivered with 6.1.0 (PIM & IGMP)
* REST API: introduced with 6.1.0 (only a small list of CRUD operations supported)
* ASA Migration Utility: introduced with 6.2.0
* FlexConfig: Due to time constraints features like PBR, BFD, EIGRP, IS-IS, VxLAN, MPF and Netflow Export were introduced in 6.2.0 using FlexConfig. For anyone who ever used CSM - its basically writing CLI config in the FMC UI and pushing it to lina on an FTD device
* On-box management: Cisco created a new UI (Firepower Device Manager) in 6.1 which includes only a subset of features compared to FMC. It is only recommended for small deployments ( < ASA 5555-X) at the moment but be aware that many features are not working if you manage your device using FDM
* ASA migration utility: Introduced with 6.2.0 (support to migrate ACLs, NAT and objects).
* Inter-Chassis Clustering: Introduced with 6.2.0
* Client-to-Site VPN: Will be introduced with 6.2.1 (May) for FP2000 appliances. All other platform will need to wait until summer
* Multiple Context Mode: A high priority item on the roadmap without a definite release date

Many features were delivered, but mostly with some drawbacks when in comes to configuration options (site-to-site vpn), manageability (flexconfig), feature parity (rest api, on-box management) and most importantly software quality.

## Software Quality

Cisco was very ambitious to satisfy the need for ASA feature parity but that ambition has lead to some very major software quality issues. At some point I have stopped counting how many cases I had to open for FTD. Most cases had to be escalated to engineering to either find a workaround or file a bug since the issue needed a hotfix. There was a variety of cases but lets just go over some examples that I found to be issues for many people out there.

### Configuration Deployment

After you make any changes to a device specific configuration within FMC you will need to deploy your configuration. The deployment process is somewhat complicated since there are many moving parts in the background. First there is configuration that needs to be applied to the snort part of FTD which works mostly fine, but then there is also the lina (asa) configuration. After I encountered the first deployment issues I started troubleshooting and found that they re-used the CSM code to deploy configuration from FMC to the lina part of an FTD device. What the deployment process basically does is get information from the FMC database and create CLI commands that are dumped into lina for configuration using CSM. If the FMC dataset is corrupt or input validation has not been implemented correctly on FMC you will see the deployment process failing... and believe me, it happens a lot. If a line of configuration throws an error the deployment process will fail and do a rollback.

I have seen various reasons for the lina configuration deployment to fail and think most of them could have been avoided if there was just better input validation on FMC (e.g. you could configure syslog options in the platform settings but if they were pushed to an FTD device it did not accept them) and a better approach to interface with lina. The deployment process feels a lot like network automation without APIs, just dump some commands into a device, do some screen scraping and cross your fingers that it will work correctly.


### High Availability

Whenever you need to replace a FTD device within an HA pair you will need to break your HA and re-build it. Unfortunately this process is destined to fail. I have had many issues where HA could not be formed correctly, FMC just applied the HA config to one device and ignored the other one, or deployment straight out failed... Or the HA removal was only being applied to one of two devices. Now the primary was standalone without HA configuration and the secondary FW just became active, because the primary had its HA configuration removed... split brain at its best. :)

### User Identity

Actually this has nothing to do with FTD but just to showcase that its not just the FTD code that needs quality improvements I thought adding user identity as an example would be a good idea. I think I have seen nearly every possible issue possible when it comes to user identity. Lets just think of the scenario where you gather user information using ISE and then use pxGRID to interface with FMC. Problems that I have encountered so far include

* User:IP mapping updates not being pushed from ISE to FMC
* User:IP mapping updates not being pushed from FMC to FTD
* User database on FMC becoming corrupt, because the FMC data purge hasn't been implemented properly
* pxGRID Failover on FMC to secondary pxGRID node not working
* etc. etc.

UPDATE 2017-07-27: In the last 6 months some major bugs considering ISE and Firepower have been fixed to enhance the stability of user identity (pxGRID failover, pxGRID consistency, etc.)

## Future Direction

I know some of this sounds like a big rant but don't get me wrong, there are some very good reasons to use firepower. The detection rate is top notch and the ability to drill down into events and use context to counter threats is something not many (or any) vendor can offer at the moment. Integration with the other products like ISE and AMP for Endpoints make the overall solution to so much more than just another middle box.

I have questioned a few times if Cisco was really committed to solve the current issues and it looks like they are finally getting the grip on this. During this weeks SEVT in San Jose they  recognized that these issues are very important and need to be fixed. The roadmap has been revamped to focus on code quality and fix current issues. To back this up they have openly talked about how they will adapt their internal practices, which was nice to see from a partners perspective.

Some major problem areas that are being addressed in the near future will probably include the following core features:

* Policy Deployment 
* FTD & FMC HA
* Clustering
* Upgrades 
