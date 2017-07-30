## PoC environment construction tutorial part4

### Introduction
It is the tutorial about installing openstack for PoC environment.<br/>
The part4 is describing mainly Cinder installing and configuration.

<[The Part1 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part1.md)><br/>
<[The Part2 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part2.md)><br/>
<[The Part3 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part3.md)><br/>

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

You must be done all tasks of following links

* [part1](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part1.md)
* [part2](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part2.md)
* [part3](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part3.md)

#### Step1: Installing and configuring the Cinder (Block Storage service) on Controller node

These tasks were executed on the controller0 node.

* Creating the database with mariadb

```sql
CREATE DATABASE cinder;
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'poc#pass';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'poc#pass';
FLUSH PRIVILEGES;
```

* Adding the credentials and defining required entities

```bash
$ source ~/admin-openrc

-- creating the cinder user
$ openstack user create --domain default --password 'poc#pass' cinder
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 2936558a7e6c47e7b512be6dbc9bce45 |
| name                | cinder                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- cinder user add to admin role
$ openstack role add --project services --user cinder admin

-- create the two cinder services
$ openstack service create --name cinderv2 --description "OpenStack Block Storage v2" volumev2
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage v2       |
| enabled     | True                             |
| id          | 269437aa5c524fa6bd12dd9cd7578675 |
| name        | cinderv2                         |
| type        | volumev2                         |
+-------------+----------------------------------+

$ openstack service create --name cinderv3 --description "OpenStack Block Storage v3" volumev3
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage v3       |
| enabled     | True                             |
| id          | 870290b8cc4e4f41b4ebdb2c33f1baf6 |
| name        | cinderv3                         |
| type        | volumev3                         |
+-------------+----------------------------------+

-- adding the API endpoints

--- volumev2
$ openstack endpoint create --region RegionOne volumev2 public http://controller0:8776/v2/%\(project_id\)s
$ openstack endpoint create --region RegionOne volumev2 internal http://controller0:8776/v2/%\(project_id\)s
$ openstack endpoint create --region RegionOne volumev2 admin http://controller0:8776/v2/%\(project_id\)s

--- volumev3
$ openstack endpoint create --region RegionOne volumev3 public http://controller0:8776/v3/%\(project_id\)s
$ openstack endpoint create --region RegionOne volumev3 internal http://controller0:8776/v3/%\(project_id\)s
$ openstack endpoint create --region RegionOne volumev3 admin http://controller0:8776/v3/%\(project_id\)s
```

* Installing cinder packages with yum

```bash
# yum install openstack-cinder
```

* Edit the /etc/cinder/cinder.conf file as follows

```ini
[DEFAULT]
transport_url = rabbit://openstack:poc3pass@controller0
auth_strategy = keystone
my_ip = 172.16.9.150
[backend]
[barbican]
[brcd_fabric_example]
[cisco_fabric_example]
[coordination]
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://cinder:poc#pass@controller0/cinder
[fc-zone-manager]
[healthcheck]
[key_manager]
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = cinder
password = poc#pass
[matchmaker_redis]
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[oslo_reports]
[oslo_versionedobjects]
[profiler]
[ssl]
```

* Populateing the cinder database

```bash
# su -s /bin/sh -c "cinder-manage db sync" cinder
```

* Edit the /etc/nova/nova.conf file and restart the nova-api service to take effect

```ini
...snip...
[cinder]
os_region_name = RegionOne
...snip...
```
Restart the nova-api service

```bash
# systemctl restart openstack-nova-api
```

* Enabling and starting the cinder serivces

```bash
# systemctl enable openstack-cinder-api openstack-cinder-scheduler
# systemctl start openstack-cinder-api openstack-cinder-scheduler
```

#### Step2: Installing and configuring the Cinder (Block Storage service) on Block node

This section's tasks were executed on the block1 node.

```bash

```