## PoC environment deployment memo

### Introduction
It is the tutorial about installing openstack for PoC environment.

### 1. Environment information
:star: The nodes are virtual machines based on KVM.
* OpenStack version: Ocata
* OS: CentOS 7.3
* Deployment method: RDO
* Server Specifications (total: 5 nodes):

Nodes|CPU|Memory|NICs|Storages
-|-|-|-|-
controller x1 | Core x4| 4 GiB| NIC x1| 50 GB
compute x2    | Core x4|4 GiB| NIC x3|50 GB
network x1    |Core x2|4 GiB |NIC x3 | 30 GB
block x1      |Core x2|4 GiB |NIC x2 | 30 GB (+secondary volume 30 GB)

* IPs and interfaces table

Hostname|Components|IPs|Interfaces
--------|----|--|---------
controller0.host.local|Horizon<br/>Keystone<br/>Glance<br/>Management APIs<br/>MariaDB<br/>RabbitMQ<br/>Memcached|eth0: 172.16.9.150|eth0
compute1.host.local|Nova|eth0: 172.16.9.151<br/>eth1: 192.168.9.151|eth0<br/>eth1<br/>eth2<br/>
compute2.host.local|Nova|eth0: 172.16.9.152<br/>eth1: 192.168.9.152|eth0<br/>eth1<br/>eth2<br/>
network1.host.local|Neutron|eth0: 172.16.9.153<br/>eth1: 192.168.9.153|eth0<br/>eth1<br/>eth2<br/>
block1.host.local|Cinder|eth0: 172.16.9.154<br/>eth1: 192.168.9.154|eth0<br/>eth1
 
### 2. Logical nodes diagram

### 3. Installation steps

Related references: [OpenStack Docs](https://docs.openstack.org/)

#### Step0: Preparation for installation

* Installing the OpenStack RPM repository and tools on all nodes
```bash
# yum install centos-release-openstack-ocata python-openstackclient
```
* Upgrading the system
* Ensuring a time synchronization on all nodes
* Gaining the privileged permission, such as root
* All password is same as 'poc#pass' on the all components
* Installing, configuring and enabling the MariaDB, RabbitMQ and Memcached on the controller node

```bash
# yum install mariadb mariadb-server python2-PyMySQL rabbitmq-server memcached python-memcached

-- MariaDB configuration
# cat /etc/my.cnf.d/00_openstack.cnf

[mysqld]
bind-address = 172.16.9.150
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

-- Enabling the MariaDB service
# systemctl enable mariadb; systemctl start mariadb

-- Configuring root account password
# mysql_secure_installation

-- RabbitMQ configuration
# systemctl enable rabbitmq-server; systemctl start rabbitmq-server

-- Add the messaging server account 'openstack'
# rabbitmqctl add_user openstack 'poc#pass'

-- Permit configuration, write, and read access for the openstack user
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"

-- Memcached configuration
# cat /etc/sysconfig/memcached
...snip...
OPTIONS="-l 127.0.0.1,::1,172.16.9.150"
...snip...

# systemctl enable memcached; systemctl start memcached

```
* Stopping the NetworkManager service on all nodes
* Stopping the Firewalld and SELinux for proceeding smoothly
* You must be resolve the node hostnames from /etc/hosts file, as follows:

```bash
# adding at the /etc/hosts on all nodes

172.16.9.150    controller0.host.local controller0
172.16.9.151    compute1.host.local    compute1
172.16.9.152    compute2.host.local    compute2
172.16.9.153    network1.host.local    network1
172.16.9.154    block1.host.local      block1
```
Let's start out after rebooting all nodes.



