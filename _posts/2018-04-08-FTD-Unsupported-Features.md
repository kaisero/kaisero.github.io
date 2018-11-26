---
layout: post
title:  "FTD Feature Limitations"
date:   2018-04-06
tags: [firepower, fmc, fdm, features, unsupported]
---

Since I see this topic coming up every now and then I thought it would be a good idea to create an overview of unsupported features for FTD in comparison to ASA. 

<!--more-->

Please keep in mind that this list is best-effort and might not reflect the current status entirely. In case you spot any errors just let me know and I will change the list accordingly.

## Unsupported

* Multiple-Context mode
* Clientless SSL VPN
* Configuration CLI
* HA (Active/Standby) for Public Cloud (AWS/Azure)
* Different vm form factors (ASAv supports various throughput options, NGFWv is capped at max 1.2Gbit)
* ASA5585-X Platform  support (not possible due to hardware architecture)
* Hyper-V support
* TLS Proxy for Encrypted Voice Inspection

## Supported with limitations

* Local device manager (no feature parity between FDM and FMC)
* Central management via in-band data path (Staging or OOB required for remote management)
* AnyConnect (no feature parity with ASA)
* REST API (no feature parity with ASA REST API yet)
* SSL Acceleration (only for FPR4100 & FPR9300)
* Clustering (only for FPR4100 & FPR9300)
* Unified Connection Logging (FTD Connection events do not include detailed L4 information, e.g. SYN Timeout, etc.)

## Supported with FlexConfig

* Modular Policy Framework (e.g. changing tcp timeouts, changing inspections depending on ACL)
* Bidirectional Forwarding Detection (BFD)
* Web Cache Communications Protocol (WCCP)
* Virtual Extensible LAN (VXLAN)
* Intermediate System to Intermediate System (IS-IS)
* Enhanced Interior Gateway Routing Protocol (EIGRP)
* Policy-based Routing (PBR)
* Equal-cost multi-path routing (ECMP)
* NetFlow



