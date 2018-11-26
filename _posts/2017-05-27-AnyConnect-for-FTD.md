---
layout: post
title:  "AnyConnect for FTD"
date:   2017-05-27
tags: [ftd, anyconnect, licensing, limitations, release, fp2100]
---

AnyConnect has been a high priority roadmap item for Firepower Threat Defense and was planned to be released in version 6.2.1 with the new Firepower 2100 appliances in april. After some delays 6.2.1 was released on the 15th of May and firepower 2100 orders started shipping. So were are we standing at the moment? What platform support AnyConnect with FTD and what features are really working?

<!--more-->

## Supported Platforms

6.2.1 should have brought AnyConnect to all hardware platform running FTD but due to unknown reasons 6.2.1 was only  released for the 2100 series of appliances and all other platforms like ASA 5500-X, FP4100 and FP9300 will need to wait until the next minor release. I think this decision was made due to stability / testing concerns, but since Cisco marketed the 2100 series with AnyConnect heavily they had to deliver something in time. I am actually happy that they did not release AnyConnect for all platforms and hope they will get AnyConnect production ready when it will be released for all platforms.

## Licensing

Smart licensing is required for AnyConnect. In case you are going through a migration from classic ASA licensing you will have to contact licensing support to change your classic license to a smart license. The licensing model of AnyConnect 4 is exactly the same for FTD. It is still a trust based license, so there is no enforcement from a technical point.

Keep in mind that AnyConnect will not work in evaluation mode since a strong crypto license is required. Also if you want to deploy remote access configuration you wont be able to  if the device is not  licensed.

## Feature Set

6.2.1 only brings a subset of AnyConnect functionality to FTD. We will see more features in upcoming releases but as of now the following features are supported: 

* Configuration using FMC and FDM
* RAVPN configuration wizard 
* Client based only (Clientless SSL/VPN is not part of the RAVPN and will be rewritten probably)
* IPSec and TLS transport 
* AD, LDAP and RADIUS Authentication
* Client Certificate + AAA authorization
* Dynamic Access Lists and time ranges
* Group Policy (Split Tunneling, access hour, timeouts, etc.)

The feature set is rather basic but will be enough to handle most of AnyConnect installations imo.

## Limitations

There are some limitations as of now with the AnyConnect implementation for FTD. This list is by no means complete but should give you a good overview on what exactly is not working at the moment.

*  VPN load-balancing cluster
*  SAML SSO
*  Double authentication
*  Local authentication 
*  WSA integration (WSA secure mobility)
*  SCEP proxy
*  Per-app VPN for mobile
*  Host Scan
*  Dynamic Acces Policy
*  ISE posture scan
*  AnyConnect Profile editor (you will need to edit anyconnect profiles using the client app)

## My recommendation

Before starting a project that involves AnyConnect on FTD make sure all the features you require are currently supported and that you test the functionality in the lab before deploying into production. As of now I would not recommend any large scale AnyConnect deployments with FTD since the features was just released a few weeks ago so I am not sure if the software quality is were it should be... Atleast the public bugtracker for firepower is very quite about AnyConnect so that might be a good sign. 
