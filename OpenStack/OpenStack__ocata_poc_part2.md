## PoC environment construction tutorial part2

### Introduction
It is the tutorial about installing openstack for PoC environment.<br/>
The part2 is describing mainly network and block nodes installation.

<[The Part1 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part1.md)>

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

#### Step0: Prerequisites the following installation

You must be done all tasks in the [part1](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part1.md).

#### Step1: Installing and configuring the Neutron (Networking service)

* Creating the database for Neutron service

```sql
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'poc#pass';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'poc#pass';
FLUSH PRIVILEGES;
```

* Defining the service and credentials

```bash
$ source ~/admin-openrc
-- Create the user
$ openstack user create --domain default --password 'poc#pass' neutron
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | dc6bbb73167c488497dc7b9aa9e87a01 |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- Adding the user to admin role
$ openstack role add --project services --user neutron admin

-- Create the service
$ openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | b7551a74442546fe848f776bd9bf6678 |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+

-- Register each API endpoints (skipped the result output)
$ openstack endpoint create --region RegionOne network public http://controller0:9696
$ openstack endpoint create --region RegionOne network internal http://controller0:9696
$ openstack endpoint create --region RegionOne network admin http://controller0:9696
```

* Installing the packages through yum on the controller0.host.local node

```bash
# yum install openstack-neutron openstack-neutron-ml2
```
