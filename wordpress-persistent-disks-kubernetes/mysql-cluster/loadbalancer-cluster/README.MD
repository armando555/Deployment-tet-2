# Setting HAproxy as load balancer for the sql cluster
## Installation

HAProxy is available in the software repositories of most Linux distributions and it’s the ports tree of FreeBSD. You can install it using the appropriate package manager.
```
apt-get install haproxy
```
## Configuration

Configuration options for HAProxy are managed through an haproxy.cfg configuration file. The above package installations generally put this file in the /etc/haproxy/ directory. However, it may have a different path depending on your operating system distribution.

To configure HAProxy to work with Galera Cluster, add the lines to the haproxy.cfg configuration file similar to the following:

```
# Load Balancing for Galera Cluster
listen galera 192.168.1.10:3306
     balance source
     mode tcp
     option tcpka
     option mysql-check user haproxy
     server node1 192.168.1.1:3306 check weight 1
     server node2 192.168.1.2:3306 check weight 1
     server node3 192.168.1.3:3306 check weight 1
```
## Enabling Database Server Checks

In addition to routing TCP connections to Galera Cluster, HAProxy can also perform basic health checks on the database server. When enabled, HAProxy attempts to establish a connection with the node and parses its response, or any errors, to determine if the node is operational.

For HAProxy, you can enable this through the mysql-check option. However, it requires that you also create a user in the cluster for HAProxy to use when connecting.
```
CREATE USER 'haproxy'@'192.168.1.10';
```

and start the server with 
```
systemctl start haproxy
```