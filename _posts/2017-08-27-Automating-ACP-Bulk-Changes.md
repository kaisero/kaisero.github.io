---
layout: post
title:  "Automating ACP Changes"
date:   2017-08-27
tags: [firepower, fmc, access-control-policy, acp, 6.2.0.2, rest, api, automation]
---

In the last few months I have found many people on the cisco support forums asking for a way to do bulk changes to their access control policy in FMC. Since the UI does not provide this functionality we can make use of the REST API of FMC to accomplish this task

<!--more-->

## Requirements

I mostly use python for automation task. I have created a small wrapper for the FMC API which is available at [github](https://github.com/kaisero/fireREST). In this post we will use the fireREST utility to interface with the FMC REST API. Make sure you install all dependencies listed in requirements.txt.

In the following example I have used FMC version 6.2.0.2 and Python 3.6. In case you encounter any API errors make sure your FMC is atleast version 6.2.0 since there have been a few bugs in the API in version 6.0.x / 6.1.x...

## Updating Log Settings

One of the things many people ask for is logging configuration for all rules. It is a very typical scenario... A syslog server is being installed and the firewall system should log all rule matches to the new system. Now imagine you have 500 rules and want to accomplish this task using the UI. That's why we will use a small and simple python script to accomplish this task.

### Step 01: Clone git repository, create a new python file and install required dependencies

Clone git repository to a directory of your choice

```
git clone https://github.com/kaisero/fireREST.git
```

```
Cloning into 'fireREST'...
remote: Counting objects: 19, done.
Receiving objects:  47% (9/19)
Receiving objects: 100% (19/19), 20.18 KiB | 0 bytes/s, done.
Resolving deltas: 100% (5/5), done.
Checking connectivity... done.
```

Change to fireREST directory, create file `example.py` and use your favorite editor to open example.py
```
cd fireREST
touch example.py
vim example.py
```

Install dependencies using pip

```
pip install -r requirements.txt
```

### Step 02: Add content to example.py

Add the following content to your example.py and edit the variables at the top to match your environment

```
# import required dependencies
from __future__ import print_function
from fireREST import FireREST

# set variables for execution. Make sure your credentials are correct, ACP exists and syslog alert definition exists
loglevel = 'DEBUG'
device = 'fmc.example.com'
username = 'admin'
password = 'Cisco123'
domain = 'Global'
policy = 'lab-policy'
syslog_alert = 'syslog-server'

# Initialize a new api object
api = FireREST(device=device, username=username, password=password, loglevel=loglevel)

# Get IDs for specified acp and syslog alert. API PK = UUID, so we have to find the matching api object for the name
# specified.

acp_id = api.get_acp_id_by_name(policy)
syslog_alert_id = api.get_syslogalert_id_by_name(syslog_alert)

# Get all access control rules for the access control policy specified
acp_rules = api.get_acp_rules(acp_id, expanded=True, domain=domain)

# Loop through HTTP response objects
for response in acp_rules:
    # Loop through access control rules in http response object
    for acp_rule in response.json()['items']:
        # Only change syslog settings if no syslog alert configuration exists
        if 'syslogConfig' not in acp_rule:
            # Get the existing rule configuration
            payload = acp_rule
            # Set syslog configuration
            payload['syslogConfig'] = {
                    'id': syslog_alert_id
            }
            # Remove metadata fields from existing rule. This is required since the API does not support
            # PATCH operations as of version 6.2.1 of FMC. Thats why we have to delete metadata before we use a PUT operation to change our ACP rule
            del payload['metadata']
            del payload['links']
            print('Trying set syslog server for rule {0} in policy {1}...'.format(acp_rule['name'], policy))
            # Send json payload to FMC REST API
            result = api.update_acp_rule(policy_id=acp_id, rule_id=acp_rule['id'], data=payload, domain=domain)
            # Verify that the PUT operation has been successful
            if result.status_code == 200:
                print('[SUCCESS]')
            else:
                print('[ERROR] Could not update syslog settings.')
        else:
            print('Syslog configuration for rule {0} in policy {1} already exists. Skipping.'.format(acp_rule['name'], policy))
```

### Step 03: Execute

When executing the script you should see the status for each rule update on your shell. In case you encounter any errors try to follow the stack trace to the line where the execution fails.

```
python example.py
```

```
Trying set syslog server for rule permit-dns in policy lab-policy...
[SUCCESS]
Trying set syslog server for rule permit-ntp in policy lab-policy...
[SUCCESS]
```
