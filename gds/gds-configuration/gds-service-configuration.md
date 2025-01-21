# GDS Configuration for Global Services

Once databases are ready to configure with GDS as per previous lab, now its time to configure GDS for Global services.

*Estimated Time*:  30 minutes

### Objectives

In this lab, you will:

* Use GDSCTL to configure DBs and GSMs for creating Global services to use by Application.
* Verify GSM configuration by creating new sqlplus connection(s) using global service(s).

### Prerequisites

This lab assumes you have:
* A Free Tier, Paid or LiveLabs Oracle Cloud account
* You have completed:
    * Lab: Prepare Setup (Free-tier and Paid Tenants only)
    * Lab: Environment Setup
    * Lab: Initialize Environment
    * Lab: GDS Install
    * Lab: Database updates to enable GDS

## Task 1: Connection to Podman instance of GSM1

## Task 1: Check for containers in your VM

1. Run the below command on terminal window that is logged in as  as **oracle** user.

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

   ```
   cat /etc/hosts
127.0.0.1       localhost.localdomain           localhost
10.0.20.100     gsm1.example.com                gsm1 
10.0.20.101     gsm2.example.com                gsm2
10.0.20.102     catalog.example.com             catalog
10.0.20.103     primary.example.com             primary
10.0.20.104     standby.example.com             standby
10.0.20.105     appclient.example.com           appclient
   ```


## Task 2: Configure GDS with Databases, create global service and validation

```
<copy>
sudo su - oracle
gdsctl
set gsm -gsm gsm1
# check connectivity to primary catalog
connect gsmcatuser/<pwd>@(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=catalogHost)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=catalogServiceName)));

example:
connect gsmcatuser/Oracle_23ai@catalog.example.com:1521/CAT1PDB;

# check connectivity to primary Data DB
connect gsmuser/<pwd>@(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=primaryDataDBHost)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=primaryDataDBServiceName)));

example:
connect gsmuser/Oracle_23ai@primary.example.com:1521/ORCLPDB1;

# check connectivity to standBy Data DB
connect gsmuser/<pwd>@(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=standByDataDBHost)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=standByDataDBServiceName)));

example:
connect gsmuser/Oracle_23ai@standby.example.com:1521/ORCLPDB1;

# re-connect to primary catalog
connect gsmcatuser/<pwd>@(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=catalogHost)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=catalogServiceName)));

example:
connect gsmcatuser/Oracle_23ai@catalog.example.com:1521/CAT1PDB;

configure -driver oci

configure -show

create gdscatalog -database "(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=catalogHost)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=catalogServiceName)))" -user gsmcatuser/<pwd> -region region1 -configname gds01 -autovncr off

example:
create gdscatalog -database "(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=catalog.example.com)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=CAT1PDB)))" -user gsmcatuser/Oracle_23ai -region region1 -configname gds01 -autovncr off

add gsm -gsm gsm1 -catalog "(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=catalogHost)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=catalogServiceName)))" -region region1 -pwd <pwd>

example:
add gsm -gsm gsm1 -catalog "(DESCRIPTION=(CONNECT_TIMEOUT=90)(RETRY_COUNT=50)(RETRY_DELAY=3)(TRANSPORT_CONNECT_TIMEOUT=3)(ADDRESS_LIST=(LOAD_BALANCE=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=catalog.example.com)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=CAT1PDB)))" -region region1 -pwd Oracle_23ai

configure -save_config

start gsm



add invitednode catalogPrimaryDB_node1_ip_address
add invitednode AppPrimaryDB_node1_ip_address
add invitednode AppPrimaryDB_node2_ip_address

#if Catalog StandBy DB exists
add invitednode catalogStandByDB_node1_ip_address
add invitednode catalogStandByDB_node2_ip_address

#if APP StandBy DB exists
add invitednode AppStandByDB_node1_ip_address
add invitednode AppStandByDB_node2_ip_address

example:
add invitednode 10.0.20.102
add invitednode 10.0.20.103
add invitednode 10.0.20.104

config gsm
# you may have to exit & return to gdsctl before running the next command
status gsm
validate
config

#App Primary Database
add database -connect AppPrimaryDBHostName:1521/AppPrimaryDBCDBServiceName -pwd <pwd>> -region region1 -gdspool dbpoolora

example:
add database -connect 10.0.20.103:1521/PORCLCDB -pwd Oracle_23ai -region region1 -gdspool dbpoolora

#Add StandBy Database
add database -connect AppStandByDBHostName:1521/AppStandByDBCDBServiceName -pwd <pwd>> -region region1 -gdspool dbpoolora

example:
add database -connect 10.0.20.104:1521/SORCLCDB -pwd Oracle_23ai -region region1 -gdspool dbpoolora

#Add  Service
add service -gdspool dbpoolora -service gds01_rw_srvc_1 -preferred_all -role primary -pdbname ORCLPDB1

start service -service gds01_rw_srvc_1
exit

#verify the GDS configuration
gdsctl
status service
services
status gsm
validate

config

```
    GDSCTL> config

    Regions
    ------------------------

    region1                       

    GSMs
    ------------------------

    gsm1                          

    GDS pools
    ------------------------

    dbpoolora                     

    Databases
    ------------------------

    porclcdb                      
    sorclcdb                      

    Services
    ------------------------

    gds01_rw_srvc_1               

    GDSCTL pending requests
    ------------------------
    Command                       Object                        Status                        
    -------                       ------                        ------                        

    Global properties
    ------------------------

    Name: gds01
    Master GSM: gsm1
    DDL sequence #: 0
```

config service
databases
exit

#TODO:Add brokerconfig
#add brokerconfig -connect AppPrimaryDBHostName:1521/AppPrimaryCDBServiceName -region region1 -gdspool dbpoolora

#example:
# Verify connectivity from the catalog's local service
sqlplus gsmcatuser@gsm1HostName:1522/GDS\$CATALOG.gds01;
example: sqlplus gsmcatuser/Oracle_23ai@gsm1.example.com:1522/GDS\$CATALOG.gds01;

# Verify connectivity from the global service to be used by App
sqlplus gsmuser/Oracle_23ai@gsm1HostName:1522/gds01_rw_srvc_1.dbpoolora.gds01;

example: sqlplus gsmuser/Oracle_23ai@gsm1.example.com:1522/gds01_rw_srvc_1.dbpoolora.gds01;

</copy>
```

### Note: How to connect Application APIs using JDBC URL:

jdbc:oracle:thin:@(DESCRIPTION=(FAILOVER=ON)(CONNECT_TIMEOUT=5)(TRANSPORT_CONNECT_TIMEOUT=3)(RETRY_COUNT=3)(ADDRESS_LIST=(ADDRESS=(HOST=gsm1HostName)(PORT=1522)(PROTOCOL=tcp)))(CONNECT_DATA=(SERVICE_NAME=gds01_rw_srvc_1.dbpoolora.gds01)))

This Global Service's JDBC URL can be shared with Application team for using it from application. If they still would like to connect Catalog directly, then share them connection for the catalog's local service.

This completes GDS configuration tasks and its verification by new db connection(s) created using Global service(s).

You may now **proceed to the next lab**.
