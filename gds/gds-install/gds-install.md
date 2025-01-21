# GDS Installation


## GDS Installation Overview

Global Data Services (GDS) installation can be done either before or after preparing the databases. We use terms GSM (Global Services Manager) to manage via Global Data Services Control Utility (GDSCTL).

*Estimated Time*:  30 minutes

### Objectives

In this lab, you will:

* Prepare the GDS container before install.
* Run GDS install.
* Verify GDS install is 100% Done and we can start using GDSCTL.

### Prerequisites

This lab assumes you have:
* A Free Tier, Paid or LiveLabs Oracle Cloud account
* You have completed:
    * Lab: Prepare Setup (Free-tier and Paid Tenants only)
    * Lab: Environment Setup
    * Lab: Initialize Environment

Additional Pre-requisites already taken care in this Live lab env but those may be needed for non-Live-lab GDS env as below:

### 1. GDS uses three Ports and those must be opened.
    the listener port   (default 1522)
    the local ons port  (default 6123)
    the remote ons port (default 6234)

You can use different ports of your choices.

### 2. Make sure that same TimeZone is used in all machines/containers for GDS and/or Databases.

e.g., UTC is used for Database, but GSM is using GMT. in such cases, make GSM machine to use UTC as well.

```
sudo su - oracle
vi ~/.bashrc
#add below
export TZ=UTC
#save, exit and recheck $date on each host as same.
[oracle@gsmhost ~]$ date
#it must me in UTC (same as Database timeZone)
```

### 3.  all Databases with same version and same Character set to be used.

## Task 1: Check for containers in your VM

1. Run the below command on terminal window that is logged in as **oracle** user.

    ```
    <copy>
    sudo podman ps -a
    </copy>
    ```

    ![<podman containers>](./images/t1-podman-containers-1.png " ")

2. From a terminal connect to **GSM1**.

   ```
   <copy>
   sudo podman exec -i -t gsm1 /bin/bash
   </copy>
   ```

## Task 2: GSM Pre-install steps

```
<copy>
# Set permissions in [oracle@gsm1 ~]$
chmod 777 /u01/app/oracle
cd $ORACLE_HOME
pwd
# make sure you are in /u01/app/oracle/product/23ai/gsmhome_1
cp /opt/oracle/install/*.* .
ls -lrt
# LINUX.X64_237000_gsm.zip and 23c_gsm_install.rsp copied to /u01/app/oracle/product/23ai/gsmhome_1
unzip LINUX.X64_237000_gsm.zip
# For GDS install use its response file: 23ai_gsm_install.rsp
cat /u01/app/oracle/product/23ai/gsmhome_1/23ai_gsm_install.rsp
</copy>
```

cat "23ai_gsm_install.rsp"

```
###############################################################################
## Copyright(c) Oracle Corporation 1998,2022. All rights reserved.           ##
##                                                                           ##
## Specify values for the variables listed below to customize                ##
## your installation.                                                        ##
##                                                                           ##
## Each variable is associated with a comment. The comment                   ##
## can help to populate the variables with the appropriate                   ##
## values.							             ##
##                                                                           ##
###############################################################################

#-------------------------------------------------------------------------------
# Do not change the following system generated value. 
#-------------------------------------------------------------------------------
oracle.install.responseFileVersion=/oracle/install/rspfmt_gsminstall_response_schema_v23.0.0

#-------------------------------------------------------------------------------
# Unix group to be set for the inventory directory.
#-------------------------------------------------------------------------------
UNIX_GROUP_NAME=oinstall
#-------------------------------------------------------------------------------
# Inventory location.
#-------------------------------------------------------------------------------
INVENTORY_LOCATION=/u01/app/oracle/oraInventory
#-------------------------------------------------------------------------------
# Complete path of the Oracle Home
#-------------------------------------------------------------------------------
ORACLE_HOME=/u01/app/oracle/product/23ai/gsmhome_1

#-------------------------------------------------------------------------------
# Complete path of the Oracle Base.
#-------------------------------------------------------------------------------
ORACLE_BASE=/u01/app/oracle
```


Note: Same GDS software can be downloaded from Oracle-site https://edelivery.oracle.com/osdc/faces/SoftwareDelivery for non Live Labs env.

## Task 3: Install GSM software

```
<copy>
From [oracle@gsm1 ~]$ which can be the default diretory after "sudo podman exec -it gsm1 /bin/bash"
cd /u01/app/oracle/product/23ai/gsmhome_1
./runInstaller -silent -responseFile 23ai_gsm_install.rsp
</copy>
```

Wait till installation completes.
Verify console logs to see the message successfully install will show 100% done.

```
As a root user, run the following script(s):

	1. /u01/app/oracle/oraInventory/orainstRoot.sh

Run /u01/app/oracle/oraInventory/orainstRoot.sh on the following nodes: 
[gsm1]


Successfully Setup Software with warning(s).
Moved the install session logs to:
 /u01/app/oracle/oraInventory/logs/GSMInstallActions2025-01-15_02-13-18PM
```
 
Some warnings may exists but those can be ignored.

## Task 4: Post Install steps

1. Run orainstRoot.sh and root.sh as a root user
```
<copy>
sudo su -
# now you are in [root@gsm1 ~]#
/u01/app/oracle/oraInventory/orainstRoot.sh
</copy>
```

2. Verify the output as below:

```
Changing permissions of /u01/app/oracle/oraInventory.
Adding read,write permissions for group.
Removing read,write,execute permissions for world.

Changing groupname of /u01/app/oracle/oraInventory to oinstall.
The execution of the script is complete.
```

3. Verify the output as below that no error exists in the file below and its file size is 0 bytes:

```
ls -lrt /u01/app/oracle/oraInventory/logs/GSMInstallActions2025-01-15_02-13-18PM/oraInstall2025-01-15_02-13-18PM.err 
```

4. edit /etc/oratab to add one line as below as root user

```
<copy>
sudo su -
# now you are in [root@gsm1 ~]#
echo 'gsm:/u01/app/oracle/product/23ai/gsmhome_1:N'  >> /etc/oratab
# cat /etc/oratab to verify a new line has been added
cat /etc/oratab
exit
</copy>
```

5. Change permission from 777 to 755 for /u01/app/oracle

```
<copy>
# if not in [oracle@gsm1 gsmhome_1]$
sudo podman exec -it gsm1 /bin/bash
#update permission
chmod 755 /u01/app/oracle
</copy>
```

6. Verify path for gsm eith via

```
cat ~/.bashrc 
#or by viewing each path
[oracle@gsm1 ~]$ echo $ORACLE_HOME
/u01/app/oracle/product/23ai/gsmhome_1
[oracle@gsm1 ~]$ echo $PATH
/u01/app/oracle/product/23ai/gsmhome_1/bin:/bin:/usr/bin:/sbin:/usr/sbin
[oracle@gsm1 ~]$ echo $LD_LIBRARY_PATH
/u01/app/oracle/product/23ai/gsmhome_1/lib:/usr/lib:/lib
[oracle@gsm1 ~]$ echo $GSM_HOME
/u01/app/oracle/product/23ai/gsmhome_1
```

7. Verify /etc/hosts for each database and gsm hosts entries

```
<copy>
cat /etc/hosts
</copy>
```

## Task 6: Run GDSCTL to confirm the successful installation.

1. To see GDSCTL prompt

```
<copy>
#If you are not in [oracle@gsm1 gsmhome_1]$ run below:
sudo podman exec -it gsm1 /bin/bash

# To see GDSCTL prompt
gdsctl
</copy>
```

2. Its expected output is as below:

```
GDSCTL: Version 23.0.0.0.0 - for Oracle Cloud and Engineered Systems on Wed Jan 15 14:27:26 UTC 2025

Copyright (c) 2011, 2024, Oracle.  All rights reserved.

Welcome to GDSCTL, type "help" for information.

Warning:  GSM  is not set automatically because gsm.ora does not contain GSM entries. Use "set  gsm" command to set GSM for the session.
Current GSM is set to GSMORA
GDSCTL>
```

3. Exit form GSM1 for next lab:

```
<copy>
exit
</copy>
```

This completes GDS installation tasks. We'll do donfiguration after **Database updates for enabling GDS**

You may now **proceed to the next lab**
