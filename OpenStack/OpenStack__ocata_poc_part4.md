## PoC environment construction tutorial part4

### Introduction
It is the tutorial about installing openstack for PoC environment.<br/>
The part4 is describing mainly Cinder installing and configuration.
Additionally, show how new instance create to you on the Horizon through browser.

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
compute1.host.local|Nova|eth0: 172.16.9.151<br/>eth1: 192.168.9.151<br/>eth2: 192.168.200.151|eth0<br/>eth1<br/>eth2<br/>
compute2.host.local|Nova|eth0: 172.16.9.152<br/>eth1: 192.168.9.152<br/>eth2: 192.168.200.152|eth0<br/>eth1<br/>eth2<br/>
network1.host.local|Neutron|eth0: 172.16.9.153<br/>eth1: 192.168.9.153|eth0<br/>eth1<br/>eth2<br/>
block1.host.local|Cinder|eth0: 172.16.9.154<br/>eth1: 192.168.200.154|eth0<br/>eth1

### 2. Logical nodes diagram

![Diagram](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_diagram1.png)

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

* Preparing the storage devices based on LVM

```bash
-- install the package through yum
# yum install lvm2

-- enable and start up the related service 
# systemctl enable lvm2-lvmetad
# systemctl start lvm2-lvmetad

-- create the physical volume and volume group with /dev/vdb (additional 30GB disk)
# vgcreate vg_cinder /dev/vdb
  Physical volume "/dev/vdb" successfully created.
  Volume group "vg_cinder" successfully created
  
-- Edit the /etc/lvm/lvm.conf file to limit used disk devices
# cat /etc/lvm/lvm.conf
devices {
...snip...
       filter = [ "a/vda/", "a/vdb/", "r/.*/"]
...snip...

```

* Installing the cinder services on the block1 node with yum

```bash
# yum install openstack-cinder targetcli python-keystone
```

* Edit the /etc/cinder/cinder.conf file as follows

```ini
[DEFAULT]
transport_url = rabbit://openstack:poc3pass@controller0
auth_strategy = keystone
my_ip = 192.168.200.154
enabled_backends = lvm
glance_api_servers = http://controller0:9292
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
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = vg_cinder
iscsi_protocol = iscsi
iscsi_helper = lioadm
```

* Enable and start up the cinder services and dependencies

```bash
# systemctl enable openstack-cinder-volume target
# systemctl start openstack-cinder-volume target
```

#### Step3: Verifying the Cinder installation (Block Storage service) on Controller0 node

Where you execute the following verifying tasks on does not matter so long as using admin credentials. 
But I had executed the commands on the controller0 node.

```bash
$ source ~/admin-openrc
$ $ openstack volume service list
+------------------+------------------------+------+---------+-------+----------------------------+
| Binary           | Host                   | Zone | Status  | State | Updated At                 |
+------------------+------------------------+------+---------+-------+----------------------------+
| cinder-scheduler | controller0.host.local | nova | enabled | up    | 2017-07-31T10:01:00.000000 |
| cinder-volume    | block1.host.local@lvm  | nova | enabled | up    | 2017-07-31T10:00:58.000000 |
+------------------+------------------------+------+---------+-------+----------------------------+
```

#### Step4: Create a new instance using installed all services through Holizon

* Login to the Horizon as pocuser

![Overview](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance1.png)

* Access the instances page and click the Launch Instance button

![Instances](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance2.png)

* Complete the launch instance format

Item|Value
-|-
Instance name | pocvm2
Image | cirros
Flavor | p2.small
Network | tenant_network1
Security Group | poc_secgroup
Key Pair | poc_keypair1

![Launch instance1](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance3.png)

![Launch instance2](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance4.png)

![Launch instance3](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance5.png)

![Launch instance4](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance6.png)

![Launch instance5](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance7.png)

![Launch instance6](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance8.png)

![Launch instance7](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance9.png)

* Allocate the other floating IP

![FloatingIP1](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance10.png)

![FloatingIP2](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance11.png)

![FloatingIP3](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance12.png)

* Create the other volume

![Volume1](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance13.png)

![Volume2](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance14.png)

![Volume3](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance15.png)

* Floating IP and volume allocate to new instance

![Assign1](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance16.png)

![Assign2](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance17.png)

![Assign3](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance18.png)

![Assign4](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_newinstance19.png)

#### Step5: Access to the new instance through SSH and mount the attached volume

