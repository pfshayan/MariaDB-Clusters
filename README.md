# MariaDB-Clusters




MariaDB Galera Cluster

MariaDB Galera Cluster is a high-availability and high-reliability database clustering technology built on top of MariaDB, which is a popular open-source relational database management system. Galera Cluster extends MariaDB's capabilities by providing synchronous multi-master replication and automatic failover, making it suitable for mission-critical applications where uninterrupted database access is essential.

Here's an explanation of MariaDB Galera Cluster in simple terms:

1. Database Cluster: Imagine you have a team of people working together on a project. In the world of databases, a cluster is like a team of database servers (computers) working together to store and manage data. These servers are called nodes.

2. Synchronous Replication: In a typical database setup, one server (the master) handles all the write operations, while others (the slaves) replicate the data from the master but can't accept writes. In MariaDB Galera Cluster, all nodes can both read and write data, and changes made on one node are immediately copied to all other nodes. This is called synchronous replication, ensuring that data is consistent across all nodes.

3. Multi-Master: In a Galera Cluster, there's no single master server. Every node is like a master and can accept write operations. This means you can distribute the workload across all nodes, providing better performance and fault tolerance.

4. Automatic Failover: Sometimes, a database server can fail due to hardware issues or other problems. With Galera Cluster, if one node fails, the cluster automatically switches to another healthy node to keep the database accessible. This ensures high availability, meaning your database remains operational even when a server goes down.

5. Data Consistency: Galera Cluster guarantees that data changes are consistent across all nodes, so you don't end up with conflicting data. If two nodes try to change the same data simultaneously, Galera ensures they agree on the final result.

6. Load Balancing: You can use a load balancer to evenly distribute database queries and connections among the cluster nodes, ensuring efficient use of resources and improved performance.

7. High Availability: Galera Cluster provides high availability by allowing you to have multiple database nodes that can take over if one fails. This minimizes downtime and keeps your application running smoothly.

In summary, MariaDB Galera Cluster is like a team of database servers working together, all capable of reading and writing data, ensuring data consistency, automatic failover, and high availability. It's a robust solution for applications that require uninterrupted access to their databases.

A MariaDB Galera cluster requires a minimum of 3 nodes. However, one of the members of the cluster can be an arbitrator (2 nodes + 1 arbitrator). Despite not participating in data replication, the arbitrator still needs to be on a 3rd physical node.

Arbitrator Node:

An arbitrator node, in simple terms, is like a referee in a sports game. Its job is to make sure there is a clear winner and prevent a tie.
In the context of a database cluster like MariaDB Galera Cluster, the arbitrator node plays a similar role. When the database nodes in the cluster are deciding who should be in charge (the primary node), they might need help if there's a disagreement, like a tie in a game.
The arbitrator node steps in to break the tie. It doesn't store much data and doesn't do the heavy lifting like the other database nodes. Instead, it provides a decision when there's confusion among the main nodes, ensuring that the database cluster always has a clear leader, preventing any confusion or conflicts.
So, you can think of an arbitrator node as the impartial referee in a game, making sure the cluster knows who's in charge and avoiding any split-brain situations or confusion among the database nodes.


Galera Arbitrator is a separate daemon from Galera Cluster called garbd. This means that you must start it separately from the cluster.

Garbd Arbitrator Configuration

/etc/conf.d/garbd


# A space-separated list of node addresses (address[:port]) in the cluster
GALERA_NODES="192.168.0.10:4567 192.168.0.11:4567"
# Galera cluster name, should be the same as on the rest of the nodes.
GALERA_GROUP="my_mariadb_cluster"

/etc/init.d/garbd start



https://www.howtoforge.com/how-to-setup-mariadb-galera-cluster-on-ubuntu-20-04/#configure-galera-cluster

https://wiki.gentoo.org/wiki/MariaDB_Galera_Cluster#:~:text=A%20MariaDB%20Galera%20cluster%20requires,on%20a%203rd%20physical%20node.




Prerequisites
Three servers running Ubuntu 20.04.
A root password is configured on the server.



Before updating you need to update your system packages to the latest version

apt-get update -y


Install MariaDB Server:

First we need to install the MariaDB server on all nodes. You can install by using following command:

apt-get install mariadb-server -y


Once the installation has been completed, start the MariaDB service and enable them to start at system reboot:

systemctl start mariadb


Check status:

systemctl status mariadb



Next you will need to secure the mariaDB installation and set a MariaDB root password on each node, you will be asked to set a MariaDB root password. You can do this by using following command:

mysql_secure_installation







Configure Galera Cluster:

Now you will need to create a Galera configuration file on each node so that each node can communicate with each other.

On the first node, create a galera.cnf file by using following command:

nano /etc/mysql/conf.d/galera.cnf



Add the following lines:

[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://node1-ip-address,node2-ip-address,node3-ip-address"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
# Galera Node Configuration
wsrep_node_address="node1-ip-address"
wsrep_node_name="node1"


Add node1-ip and node2-ip addresses 
Save and close the file when you are finished.


On the second node, create a galera.cnf file using the following command:

nano /etc/mysql/conf.d/galera.cnf


Add the following lines:

[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://node1-ip-address,node2-ip-address,node3-ip-address"
# Galera Synchronization Configuration
wsrep_sst_method=rsync
# Galera Node Configuration
wsrep_node_address="node2-ip-address"
wsrep_node_name="node2"


Add node1-ip and node2-ip addresses.
Save and close the file when you are finished.


Initialize the Galera Cluster

At this point, all nodes are configured to communicate with each other.

Next, you will need to stop the MariaDB service on all nodes. You can use the following command to stop the MariaDB service:

systemctl stop mariadb


On the first node, initialize the MariaDB Galera cluster with the following command:

galera_new_cluster


Now, check the status of the cluster with the following command:

mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"



You should see the following output:



+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
On the second node, start the MariaDB service with the following command:

systemctl start mariadb


Next, check the status of the MariaDB Galera cluster with the following command:

mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"


You should see the following output:

+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
At this point, the MariaDB Galera cluster is initialized. You can now proceed to the next step.

Verify Cluster Replication

Next, you will need to verify whether the replication is working or not.
On the First node, connect to the MariaDB with the following command:

mysql -u root -p


Once your are connected, create some database with the following command:

create database db1;


create database db2;


Next, exit from the MariaDB with the following command:
exit;


Next, go to the second node and log in to the MariaDB with the following command:

mysql -u root -p


Next, run the following command to show all databases:
show databases;


You should see that both databases that we have created on the first node are replicated on the second node:

+--------------------+
| Database           |
+--------------------+
| db1                |
| db2                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
5 rows in set (0.001 sec)






































 













