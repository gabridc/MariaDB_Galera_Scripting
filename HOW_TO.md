# 1. Install mariadb and galera

sudo apt update
sudo apt install mariadb-server mariadb-client galera-4

# 2. First node

## 2.1 Configure first node "MASTER" of the cluster

Modify file /etc/mysql/mariadb.conf.d/50-server.cnf adding this content

[galera]
wsrep_on                 = ON
wsrep_cluster_name       = "MariaDB Galera Cluster"
wsrep_provider           = /usr/lib/galera/libgalera_smm.so
wsrep_cluster_address    = "gcomm://<IP_Address of this node>,<IPs_other_nodes>"
binlog_format            = row
default_storage_engine   = InnoDB
innodb_autoinc_lock_mode = 2
bind-address = 0.0.0.0
wsrep_node_address="<IP_Addres of this node>"

# 2.2. Start the cluster

Run command

galera_new_cluster

Check maraidb is running 

systemctl status mariadb

# 3. Other nodes

## 3.1 Configure other nodes of the cluster

Modify file /etc/mysql/mariadb.conf.d/50-server.cnf adding this content

[galera]
wsrep_on                 = ON
wsrep_cluster_name       = "MariaDB Galera Cluster"
wsrep_provider           = /usr/lib/galera/libgalera_smm.so
wsrep_cluster_address    = "gcomm://<IP_Address of this node>,<IPs_other_nodes>"
binlog_format            = row
default_storage_engine   = InnoDB
innodb_autoinc_lock_mode = 2
bind-address = 0.0.0.0
wsrep_node_address="<IP_Addres of this node>"

# 3.2. Start the node

Run command

systemctl start mariadb

Check maraidb is running 

systemctl status mariadb

# 4. Check that cluster is working

## 4.1 Create a database

sudo mysql -u root -p 
mysql> CREATE DATABASE test;

## 4.2 Check database test in other nodes

sudo mysql -u root -p 
mysql> SHOW DATABASES;


# 5. Restart cluster after shutdown nodes

## 5.1 Find the last node that was shutdown

The file /var/lib/mysql/grastate.dat has a variable safe_to_bootstrap, IF this variable is set to 1 indcates that this node was the latest.

## 5.2 Start the cluster "MASTER"

In the node with safe_to_bootstrap: 1 runs:

sudo rm /var/lib/mysql/ib_logfile1 /var/lib/mysql/ib_logfile0
galera_new_cluster

## 5.3 Start other nodes

sudo rm /var/lib/mysql/ib_logfile1 /var/lib/mysql/ib_logfile0
systemctl start mariadb

# 6. Recovery sequence

When a hard crash occurss the nodes does not know if it is the last node shutdown so the variable safe_to_bootstrap in all nodes is safe_to_bootstrap: 0.
In this case we have the tool galera_recovery to recover the last sequence register in each node, file /var/lib/mysql/grastate.dat contains variable seqNo
with this information. 

The higest seqNo indicates that this node has the latest changes donde in the databse so this node must be set MANUALLY as safe_to_bootstrap: 1
then start the cluster following steps 5.2 and 5.3. 

If node does not recover last status of the database run this command:

sudo rm /var/lib/mysql/ib_logfile1 /var/lib/mysql/ib_logfile0
galera_recovery mariadb@<IP_Master_Node>





