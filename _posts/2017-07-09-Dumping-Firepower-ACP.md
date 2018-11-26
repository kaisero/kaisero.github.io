---
layout: post
title:  "Dumping Firepower AC-Policy"
date:   2017-07-09
tags: [firepower, sensor, access-control-policy, troubleshooting, ftd]
---

Have you ever been in a situation where you wanted to verify the actual access control policy deployed to your sensor?
When I first started looking around on how to do this from a firepower sensor cli I found the following command  `show access-control-config` which displays a human readable version of the full access control policy. After some updates that misbehaved I was looking for an easy method to dump my policy before starting an upgrade so I can do a `diff` between my policy before the upgrade and after the upgrade.

<!--more-->

At first I had to find out a way to dump the `show access-control-config` output into a file so I switched to bash aka expert mode using the `expert` command on sfcli. I located the sfcli.pl binary at `/ngfw/var/sf/bin` on my FTD device and found that it referenced some libraries in SF::CLI. After some searching through the filesystem I came across all the CLI binaries at `/ngfw/Volume/6.2.0/sf/lib/perl/5.10.1/SF/CLI/` and started analyzing the `firewall.pm`. I found that I could call the `show access-control-config` command using the following sfcli parameters:

```
sfcli.pl show firewall
```

The last step was piping the output to a file so I could use `vi` and `diff` to analyze the policy

```
sfcli.pl show firewall > /var/tmp/access-control-policy.dump
```

Using bash I found a variety of use cases that help me during troubleshooting sessions

* Searching the deployed policy using `grep`

  ```
  sfcli.pl show firewall | grep <filter>
  ```

* Verifying rule live hitcount of a firewall rule using the `watch` command

  ```
  watch 'sfcli.pl show firewall | grep -A 12 "Rule\: <MY-RULE-NAME>"'
  ```

* Diffing policies after upgrades to make sure there hasn't been any bug that caused my policy to change

  ```
  diff /var/tmp/access-control-policy.dump /var/tmp/access-control-policy2.dump
  ```

Using standard linux tools to automate troubleshooting steps can be quite helpful. I will try to find more examples on how I can combine sfcli and bash to get better insights into the system and spot issues quicker... more blog posts will follow soon. :)







