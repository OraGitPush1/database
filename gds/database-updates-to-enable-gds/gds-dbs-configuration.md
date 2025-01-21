# Database updates for enabling GDS

Before configuring GDS using GDSCTL, Databases needs to be ready with some updates.

*Estimated Time*:  15 minutes

### Objectives

In this lab, you will:

* Configure each Database for commands to be run using sysdba.
* Verify Database are ready, so that we can start GDS configuration using GDSCTL in the next lab.

### Prerequisites
This lab assumes you have:
* A Free Tier, Paid or LiveLabs Oracle Cloud account
* You have completed:
    * Lab: Prepare Setup (Free-tier and Paid Tenants only)
    * Lab: Environment Setup
    * Lab: Initialize Environment


## 
## Task 1: Connection to Podman instance of Catalog Primary DB

```
<copy>
sudo podman exec -it catalog /bin/bash
</copy>
```

Apply Database Configuration for Catalog which will be used in "create gdscatalog.." step from GSM to apply GDS configuration steps later on.

```
<copy>
# vi .bashrc to update .bashrc of oracle user if not already
echo $ORACLE_SID
# it should CATCDB
echo $ORACLE_HOME/
echo $PATH
echo $LD_LIBRARY_PATH
echo $ORACLE_BASE

sqlplus / as sysdba
#select USERNAME,ACCOUNT_STATUS from dba_users where USERNAME='GSMCATUSER';
# If GSMCATUSER is Locked it needs to unlocked for Catalog db.
alter user gsmcatuser account unlock;
alter user gsmcatuser identified by &pwd;

show parameter open_links
show parameter open_links_per_instance
show parameter db_files
show parameter global_names
# update paramters as needed and not set earlier
alter system set open_links=255 scope=spfile;
alter system set open_links_per_instance=255 scope=spfile;
alter system set db_files=1024 scope=spfile;
alter system set global_names=false scope=spfile;
# set trace level to help to debug issues during install steps.
# If you want to set the GSM request traces at DB it needs to set event(s) 
show parameter event
alter system set events 'immediate trace name GWM_TRACE level 7';
alter system set event='10798 trace name context forever, level 7' scope=spfile;

shutdown immediate
startup mount;

alter database archivelog;
# If Flashback Database logging is turned off make it turn on:
alter database flashback on;
alter database open;
# If database is not in force logging mode, it can be updated by:
alter database force logging;
# to check archive details:
archive log list;

select name, force_logging, flashback_on from v$database;
# expected output
```
NAME	  FORCE_LOGGING 			  FLASHBACK_ON
--------- --------------------------------------- ------------------
CATCDB	  YES					  YES
```

show pdbs
alter session set container = CAT1PDB;
#select USERNAME,ACCOUNT_STATUS from dba_users where USERNAME='GSMCATUSER';
# If GSMCATUSER is Locked it needs to unlocked for Catalog db.
alter user gsmcatuser account unlock;
# exit twice to return to [opc@oraclegdshost:~]$
exit;
exit;
</copy>
```

## Task 2: Connection to Podman instance of Primary Database for Application

```
<copy>
sudo podman exec -it primary /bin/bash
</copy>
```

Prepare Primary Database for the application. This Database will be used in "add database" step from GSM to apply GDS configuration steps later on.

```
<copy>
sqlplus /nolog
connect / as sysdba
select USERNAME,ACCOUNT_STATUS from dba_users where USERNAME='GSMUSER' or USERNAME='GSMROOTUSER';
#if both are locked those needs to be unlocked and password setting is needed
alter user gsmrootuser account unlock;
# set password "Oracle_23ai"
alter user gsmrootuser identified by "Oracle_23ai";
grant sysdg, sysbackup to gsmrootuser;

# check Data Pump Directory path and if its null it can be set: 
select directory_path from dba_directories where directory_name='DATA_PUMP_DIR';
# create or replace directory data_pump_dir as '/opt/oracle/admin/PORCLCDB/dpdump/';
grant read, write on directory DATA_PUMP_DIR to gsmadmin_internal;



alter user gsmuser account unlock;
alter user gsmuser identified by "Oracle_23ai";
grant sysdg, sysbackup to gsmuser;
grant debug connect session to gsmuser;

show parameter db_files
show parameter dg_broker
show parameter global_names

alter system set db_files=64000 scope=spfile;
alter system set dg_broker_start=true scope=both;
alter system set global_names=false scope=spfile;

alter system set events 'immediate trace name GWM_TRACE level 7';
alter system set event='10798 trace name context forever, level 7' scope=spfile;

shutdown immediate
startup mount;
select name, log_mode, flashback_on,force_logging from v$database;
#if any og the value is not set to "YES" then run below to set it.
alter database archivelog;
alter database flashback on;
alter database force logging;

alter database open;

archive log list;

show pdbs
#select name as pdb
#from   v$containers
#where  name not in ('CDB$ROOT', 'PDB$SEED');
alter session set container = &pdb;
select USERNAME,ACCOUNT_STATUS from dba_users where USERNAME='GSMUSER';
# If GSMUSER is locked unlock it. 
alter user gsmuser account unlock;
grant sysdg, sysbackup, gsmuser_role to gsmuser;

grant read,write on directory DATA_PUMP_DIR to gsmadmin_internal;
grant read,write on directory DATA_PUMP_DIR to gsmuser;
# exit form Primary db to return to host
exit;
exit;

</copy>
```


### Task 3: Connection to Podman instance of StandBy Database for Application Data

```
<copy>
sudo podman exec -it appdb2 /bin/bash
</copy>
```

Prepare StandBy Database for the application. This Database will be used in "add database" step from GSM to apply GDS configuration steps later on.

```
<copy>
sudo su - oracle
sqlplus / as sysdba

alter user gsmrootuser account unlock;
alter user gsmrootuser identified by &pwd;
grant sysdg, sysbackup to gsmrootuser;

create or replace directory data_pump_dir as '/u01/app/oracle/oradata';
grant read, write on directory DATA_PUMP_DIR to gsmadmin_internal;

select directory_path from dba_directories where directory_name='DATA_PUMP_DIR';

alter user gsmuser account unlock;
alter user gsmuser identified by &pwd;
grant sysdg, sysbackup to gsmuser;
grant debug connect session to gsmuser;

show parameter db_files
show parameter dg_broker
show parameter global_names

alter system set db_files=64000 scope=spfile;
alter system set dg_broker_start=true scope=both;
alter system set global_names=false scope=spfile;

alter system set events 'immediate trace name GWM_TRACE level 7';
alter system set event='10798 trace name context forever, level 7' scope=spfile;

shutdown immediate
startup mount;

alter database archivelog;
alter database flashback on;

alter database open read only;

alter database force logging;

show parameter db_files
show parameter dg_broker
show parameter global_names

archive log list;

select name, force_logging, flashback_on from v$database;
show pdbs
#select name as pdb
#from   v$containers
#where  name not in ('CDB$ROOT', 'PDB$SEED');
alter session set container = &pdb;

alter user gsmuser account unlock;
grant sysdg, sysbackup, gsmuser_role to gsmuser;

grant read,write on directory DATA_PUMP_DIR to gsmadmin_internal;
grant read,write on directory DATA_PUMP_DIR to gsmuser;
</copy>
```

This completes Database tasks to get ready to configure with GDS.

You may now **proceed to the next lab**.
