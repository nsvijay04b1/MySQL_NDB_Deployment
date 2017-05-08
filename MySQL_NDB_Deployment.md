# Deployment of MySQL NDB Cluster

# Map of nodes
  - Data nodes:
```
env3 & env4
```
  - MySQL nodes:
```
env1 & env2
```
  - Management node:
```
mgmd
```
--------------------------------------------------------------


# IMPORTANT: Installing a minimal SLES_12SP2 OS 0r Other Linux OS!
-------------------------------------------------------------

   
# 1. Create directory where you will downloading the packages;
 
```
mkdir mysql
mkdir mysql/mysql-server
```
- Download the packages:

```
cd /mysql/mysql-server
wget https://downloads.mysql.com/archives/get/file/MySQL-Cluster-gpl-7.4.6-1.sles11.x86_64.rpm-bundle.tar
```
# Note: Download the packages, on all five machines!
----------------------------------------------------




# 2. Installation:
----------------------------------------------------
# For CentOS
   :: + perl package perl-Data-Dumper ::
   yum install perl-Data-Dumper


- On mgmd:
  
```
cd /mysql/mysql-server
rpm -Uvh ...package-name.rpm

```

- On env1:
  
```
cd /mysql/mysql-server
rpm -Uvh ...package-name.rpm

```

- On env2:
  
```
cd /mysql/mysql-server
rpm -Uvh ...package-name.rpm

```

- On env3:
  
```
cd /mysql/mysql-server
rpm -Uvh ...package-name.rpm

```

- On env4:
  
```
cd /mysql/mysql-server
rpm -Uvh ...package-name.rpm

```
# Note: Install all packages using "rpm -Uvh" command!
------------------------------------------------------



# 3. Configuration:

- On mgmd:
  Create configuration cluster directory and config file inside of cluster directory "config.ini" 

```
mkdir /var/lib/mycluster
touch /var/lib/mycluster/config.ini

```
  Configure your cluster:

```
vim /var/lib/mycluster/config.ini
```
:: add your configuration inside, with your own parameters!::

The file should look like that:

```
[ndb_mgmd]
hostname=000.000.000.000  (your host IP)
datadir=/var/lib/mycluster


[ndbd default]
NoOfReplicas=0            (your count number of nodes)
DataMemory=000M           (your physical memory which you have and how much you want to give for the cluster)
IndexMemory=000M          (Give an index memory for server)
ServerPort=2202           (See (AFSDSE)IMPORTANT)

[ndbd]
hostname=000.000.000.000  (your host IP) - your data node
datadir=/var/lib/mycluster-data

[ndbd]
hostname=000.000.000.000  (your host IP) - your second data node
datadir=/var/lib/mycluster-data

[mysqld]
hostname=000.000.000.000  (your host IP) - you mysql node 

[mysqld]
hostname=000.000.000.000  (your host IP) - your second mysql node
```
# (AFSDSE)IMPORTANT: If you running a firewall you should have to configure the firewall 
                     to use the ports which are depend to be opened, for the cluster!
---------------------------------------------------------------------------------

- On The data nodes:
  env3 and env4

  Create "my.cnf" config file in /etc/ directory, which telling on data node, who is the "mgmd" node!
  
  env3:
  
```
mkdir /var/lib/mycluster-data
vim /etc/my.cnf
```
add in my.cnf:

```
[mysql_cluster]
ndb-connectstring=000.000.000.000      (your "mgmd" host IP)
```

 Create "my.cnf" config file in /etc/ directory, which telling on data node, who is the "mgmd" node!
  
  env4:
  
```
mkdir /var/lib/mycluster-data
vim /etc/my.cnf
```
add in my.cnf:

```
[mysql_cluster]
ndb-connectstring=000.000.000.000      (your "mgmd" host IP)
```
------------------------------------------------------------------------------


- On Tha mysql nodes:
  env1 and env2

 
Create "my.cnf" config file in /etc/ directory for starting the mysql cluster support 
        and creation of the storage cluster table!
 
 env1:
  
```
vim /etc/my.cnf
```
add in:

```
[mysqld]
ndbcluster
default-storage-engine=NDBCLUSTER


[mysql_cluster]
ndb-connectstring=000.000.000.000      (your "mgmd" host IP)

```

    Create "my.cnf" config file in /etc/ directory for starting the mysql cluster support 
            and creation of the storage cluster table!
  env2:
  
```
vim /etc/my.cnf
```
add in:

```
[mysqld]
ndbcluster
default-storage-engine=NDBCLUSTER


[mysql_cluster]
ndb-connectstring=000.000.000.000      (your "mgmd" host IP)

```
-----------------------------------------------------------------


# 4. Preparing for starting the cluster:
# IMPORTANT: See (AFSDSE)IMPORTANT:

- On the env1 
  
```
mysqld_safe &
```
  check port status ans service:

```
netstat -nltp
ps -ef | grep mysql

```
  Use the secret password of mysql to login as root.

```
cat /root/.mysql_secret
```
  Copy the password after (local time): ........... 
  and use this password to login as root.
  if you want to change your MySQL root password:

```
mysql_secure_installation

```

  Create ".my.cnf" config file to login as root automatically.

```
vim /root/.my.cnf
```
  add in .my.cnf

```
[client]
user=root
password=...your password...
```


- On the env2 
  
```
mysqld_safe &
```
  check port status ans service:

```
netstat -nltp
ps -ef | grep mysql

```
  Use the secret password of mysql to login as root.

```
cat /root/.mysql_secret
```
  Copy the password after (local time): ........... 
  and use this password to login as root.
  if you want to change your MySQL root password:

```
mysql_secure_installation

```

  Create ".my.cnf" config file to login as root automatically.

```
vim /root/.my.cnf
```
  add in .my.cnf

```
[client]
user=root
password=...your password...
```

- On mgmd 

  Start the cluster"

```
ndb_mgmd -f /var/lib/mycluster/config.ini
```

  Check depending ports and if you use firewall,
  you should to configure your firewall to allow this ports which cluster is using!
# WARNING: See (AFSDSE)IMPORTANT: if you use a firewall!

```
netstat -nltp
```
--------------------------------------------------------------
  
# 5.  Login to a cluster

- On mgmd

  Login and check:

```
ndb_mgm
```
  Use "show" command to view which node are connected

```
ndb_mgm> show
```

  You should see:

```
[ndbd(NDB)]   2 node(s)
-----------------------------------
id=2 (not connected.....................)
id=3 (not connected.....................)
 
[ndb_mgmd(MGM)] 1 node(s) 
id=1  @000.000.000.000 (your mgm host IP) (mysql-5.6.24 ndb-7.6.4)


[mysql(API)]   2 node(s)
-----------------------------------
id=4 (not connected.....................)
id=5 (not connected.....................)

```
-----------------------------------------------------------------

# 6. Starting the data nodes:

- On env3 and env4

```
ndbd
```
  You should see
# WARNING: See (AFSDSE)IMPORTANT: if you use a firewall! 

```
....[ndbd] INFO       --Angel connected to (Your mgmd host IP:1186)
....[ndbd] INFO       --Angel allocated nodeid: 2
```
  Check again to see if they are connected to the cluster.

```

- On mgmd

  Login and check:

```
ndb_mgm
```
  Use "show" command to view which node are connected

```
ndb_mgm> show
```
-----------------------------------------------------------------

# 7. Start the mysql nodes:

- On env1 and env2

```
mysqld_safe &
```

# Check again if they are not connected, run again!

```
mysqld_safe &
```
```


# Have fun with n11secur1ty =) 
