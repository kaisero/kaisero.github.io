---
layout: post
title:  "Automating FX-OS Provisioning"
date:   2017-08-27
tags: [firepower, fxos, rest, automation, provisioning]
---

In the last year I have installed a few FP 4100 and FP 9300 appliances which thought me one thing... Provisioning by hand takes too much time and should be automated to avoid inconsistent configuration and wasted hours waiting for upgrades to complete. Since a nearly feature complete REST API is available for FX-OS I started developing a small library to interface with the API and found the results to be very satisfying.

<!--more-->

## My Goal

My goal was to create a simple wrapper for the API that can be used to automate the provisioning of a new appliance including...

* Basic configuration settings for the chassis itself including...
  * Physical interface
  * PortChannel interfaces
  * DNS configuration
  * NTP configuration
  * SNMP (Trap) configuration
 
* Uploading required firmware packages and incrementally upgrading the chassis according to the supported firmware path.
* Uploading application software (FTD image) and configuring logical device settings

## FXOS REST Library

To my surprise the API guide available at  [Cisco Devnet](https://developer.cisco.com/site/ssp/firepower/) was quite useful in getting started. Unfortunately it is not a complete API reference which left me with a lot of trial and error to determine which payload format was required to make changes but since the error messages were mostly very verbose I was able to work around any issues I encountered.

The result is this python library which is available to anyone who wants to get started with working with the API without re-inventing the wheel: https://github.com/kaisero/fxosREST

## Firepower Manager

Based on the FX-OS library I created a python script to handle all the validation and configuration logic to automate a new chassis installation. It is currently a company internal tool which I will hopefully be able to publish at some point.

If anyone would be interested in a tool like that let me know. :)



