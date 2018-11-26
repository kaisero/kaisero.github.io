---
layout: post
title:  "FMC Bug - Large Backup Files"
date:   2017-08-27
tags: [firepower, fmc, backup, 6.2.0.2, bug]
---

I encountered an interesting bug in 6.2.0.2 which I would like to share with the community in case anybody else is having the same issue. On one of my FMC installations I found that the backups were rapidly growing from 2.5G to 9.5G in size. After some research and help from Cisco TAC we were able to pinpoint the issue and implement a workaround.

<!--more-->

## CSCvf54853 

Backups for FMC in HA mode were rapidly growing. Each day our backup size would linearly increase by 250MB. After checking the contents of the backup tarball we found that the `vms.db`file was constantly growing. vms.db is the sybase database file used by FMC.

Using a database query we found that the `user_group_map` table was constantly growing, causing the database size to increase and therefore causing larger backup files.

```
OmniQuery.pl -e -db sdb -e "SELECT table_name, count(*) from sym_data group by table_name order by 2 desc";
```

```
+------------------------+-----------+
| table_name             | count()   | 
+------------------------+-----------+
| user_group_map         | 157322150 | 
| user_group             | 1821817   | 
| firewall_rule_cache    | 1565817   | 
|  (...)                             |
+------------------------+-----------+
```

The cause of our issue was user identity data that is periodically polled by FMC (Active Directory LDAP Integration). At each sync the required database records would be overwritten, but metadata of the old data was still being kept in the database. Due to our large Active Directory dataset this metadata caused our database to grow without the old data getting cleaned up.

CSCvf54853 should be fixed in an upcoming release, but in case your symptoms match the problem description make sure to contact TAC to receive a fix.

