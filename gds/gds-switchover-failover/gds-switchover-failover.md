# Switchover and Fail-over tests while using Global services

Once global service(s) created along with Data guard configuration as per previous lab, now it’s time to run Switchover and Failover tests. Application continuity is expected with zero data loss during Switchover and Failover tests for connections using Global services.

Estimated Time: 20 minutes

### Objectives

In this lab, you will:

* Use DGMGRL to run and verify Switchover and Failover tests.
* Verify Database connection with global service(s).

### Prerequisites

This lab assumes you have:

* A Free Tier, Paid or LiveLabs Oracle Cloud account
* You have completed:
    * Lab: Prepare Setup (Free-tier and Paid Tenants only)
    * Lab: Environment Setup
    * Lab: Initialize Environment
    * Lab: GDS Install
    * Lab: Database updates to enable GDS
    * Lab: GDS Configuration for Global Services

## Task 1: Connect to Podman instance of Primary DB to verify current Primary and StandBy DB modes. 

```
<copy>
sudo podman exec -it primary /bin/bash
dgmgrl
</copy>
```

Verify output:

```
DGMGRL> connect /
Connected to "pdb_of_PrimaryDB"
Connected as SYSDG.
DGMGRL> show configuration

Configuration - pdb_of_PrimaryDB

  Protection Mode: MaxPerformance
  Members:
  pdb_of_PrimaryDB - Primary database
  pdb_of_StandByDB - Physical standby database 

Fast-Start Failover:  Enable with zero data loss

Configuration Status:
SUCCESS   (status updated 6 seconds ago)
```

##  Task 2: Perform switchover from primary to standBy


sudo podman exec -it primary /bin/bash
a) Start insert record requests as application continuity test before switchover
b) execute switchover_primary_to_standby.sh
c) verify switchover from primary to standby
d) Verify insert record requests as application continuity test during and till switchover completes.

##  Task 3: Perform switchback from standBy to primary


sudo podman exec -it standby /bin/bash
a) Start insert record requests as application continuity test before switch-back from standby to primary
b) execute switchback_standby_to_primary.sh
c) verify switchback from standby to primary
d) Verify insert record requests as application continuity test during and till switchover completes.


##  Task 3: Perform Fail-over from primary to standby

sudo podman exec -it primary /bin/bash
a) Start insert record requests as application continuity test before fail-over
b) execute failover_primary_to_standby.sh
c) verifyfailover from primary to standby
d) Verify insert record requests as application continuity test during and till failover completes.

##  Task 4: Perform Failback from standBy to primary


sudo podman exec -it standby /bin/bash
a) Start insert record requests as application continuity test before fail-back from standby to primary
b) execute failback_standby_to_primary.sh
c) verify failback from standby to primary
d) Verify insert record requests as application continuity test during and till failback completes.

Verify Application is running without any data loss but if there are some test cases didn't go executed as expected, review the Trouble-shooting Lab.

This Completes the GDS introduction LiveLabs.
