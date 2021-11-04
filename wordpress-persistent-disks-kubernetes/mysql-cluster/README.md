# Setting the cluster of MariaDB 
First step is installing in the 3 VM the mariadb-server with this command
```
sudo apt install mariadb-server
```
later, we have to configure the mysql
```
sudo mysql -uroot
```
```
set password = password("your_password");
```
You now have all of the pieces necessary to begin configuring the cluster, but since you’ll be relying on rsync in later steps, make sure it’s installed:
```
sudo apt install rsync
```
## Setting the first node
By default, MariaDB is configured to check the /etc/mysql/conf.d directory to get additional configuration settings from files ending in .cnf. Create a file in this directory with all of your cluster-specific directives:
```
sudo nano /etc/mysql/conf.d/galera.cnf
```
```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://First_Node_IP,Second_Node_IP,Third_Node_IP"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="This_Node_IP"
wsrep_node_name="This_Node_Name"
```
The first section modifies or re-asserts MariaDB/MySQL settings that will allow the cluster to function correctly. For example, Galera won’t work with MyISAM or similar non-transactional storage engines, and mysqld must not be bound to the IP address for localhost. You can learn about the settings in more detail on the Galera Cluster system configuration page.


The “Galera Provider Configuration” section configures the MariaDB components that provide a WriteSet replication API. This means Galera in your case, since Galera is a wsrep (WriteSet Replication) provider. You specify the general parameters to configure the initial replication environment. This doesn’t require any customization, but you can learn more about Galera configuration options.


The “Galera Cluster Configuration” section defines the cluster, identifying the cluster members by IP address or resolvable domain name and creating a name for the cluster to ensure that members join the correct group. You can change the wsrep_cluster_name to something more meaningful than test_cluster or leave it as-is, but you must update wsrep_cluster_address with the private IP addresses of your three servers.


The “Galera Synchronization Configuration” section defines how the cluster will communicate and synchronize data between members. This is used only for the state transfer that happens when a node comes online. For your initial setup, you are using rsync, because it’s commonly available and does what you’ll need for now.


The “Galera Node Configuration” section clarifies the IP address and the name of the current server. This is helpful when trying to diagnose problems in logs and for referencing each server in multiple ways. The wsrep_node_address must match the address of the machine you’re on, but you can choose any name you want in order to help you identify the node in log files.

When you are satisfied with your cluster configuration file, copy the contents into your clipboard, save and close the file. With the nano text editor, you can do this by pressing CTRL+X, typing y, and pressing ENTER.

Now that you have configured your first node successfully, you can move on to configuring the remaining nodes in the next section

## setting the remaining nodes
In this step, you will configure the remaining two nodes. On your second node, open the configuration file:
```
sudo nano /etc/mysql/conf.d/galera.cnf
```
```
. . .
# Galera Node Configuration
wsrep_node_address="This_Node_IP"
wsrep_node_name="This_Node_Name"
. . .
```
Save and exit the file.

Once you have completed these steps, repeat them on the third node.

You’re almost ready to bring up the cluster, but before you do, make sure that the appropriate ports are open in your firewall.
```
sudo ufw allow 3306,4567,4568,4444/tcp
sudo ufw allow 4567/udp
```
## starting the cluster
Stop the MariaDB in all the servers
```
sudo systemctl stop mysql
```
```
sudo systemctl status mysql
```
### bring up the first node
To bring up the first node, you’ll need to use a special startup script. The way you’ve configured your cluster, each node that comes online tries to connect to at least one other node specified in its galera.cnf file to get its initial state. Without using the galera_new_cluster script that allows systemd to pass the --wsrep-new-cluster parameter, a normal systemctl start mysql would fail because there are no nodes running for the first node to connect with.
```
sudo galera_new_cluster
```
This command will not display any output on successful execution. When this script succeeds, the node is registered as part of the cluster, and you can see it with the following command:
```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```
#### note:
On the remaining nodes, you can start mysql normally. They will search for any member of the cluster list that is online, so when they find one, they will join the cluster.
```
sudo systemctl start mysql
```
```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```