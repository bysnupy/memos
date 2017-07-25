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

#### Step1: Installing and configuring the Neutron (Networking service) on Controller node
The following tasks should be done on the controller0 node.

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

* Edit /etc/neutron/neutron.conf file as follows

```ini
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:poc3pass@controller0
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[agent]
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://neutron:poc#pass@controller0/neutron
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = neutron
password = poc#pass
[matchmaker_redis]
[nova]
auth_url = http://controller0:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = services
username = nova
password = poc#pass
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]
```

* Edit the /etc/neutron/plugins/ml2/ml2_conf.ini

```ini
[DEFAULT]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
flat_networks = external_network
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = true
```

* Create the symbolic link to /etc/neutron/plugin.ini

```bash
# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

* Edit the /etc/nova/nova.conf file on controller0 node and restart the nova-api service

```ini
..snip...
[neutron]
url = http://controller0:9696
auth_url = http://controller0:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = services
username = neutron
password = poc#pass
service_metadata_proxy = True
metadata_proxy_shared_secret = poc#pass
...snip...
```

The new settings take effect after restarting

```bash
# systemctl restart openstack-nova-api
```

* Service enabling and starting

```bash
# systemctl enable neutron-server
# systemctl start neutron-server
```

#### Step2: Installing and configuring the Neutron (Networking service) on Networking node
The following tasks should be done on the network1 node.

* Installing the packages through yum command

```bash
# yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

* Edit /etc/neutron/neutron.conf same as controller0 node

```ini
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:poc3pass@controller0
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[agent]
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://neutron:poc#pass@controller0/neutron
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = neutron
password = poc#pass
[matchmaker_redis]
[nova]
auth_url = http://controller0:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = services
username = nova
password = poc#pass
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]
```

* Edit the /etc/neutron/plugins/ml2/ml2_conf.ini same as controller0 node

```ini
[DEFAULT]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
flat_networks = external_network
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = true
```

* Edit the /etc/neutron/plugins/ml2/linuxbridge_agent.ini file as follows

```ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = external_network:eth2
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = True
local_ip = 192.168.9.153
l2_population = True
```

* Edit the /etc/neutron/l3_agent.ini as follows

```ini
[DEFAULT]
interface_driver = linuxbridge
[agent]
[ovs]
```

* Edit the /etc/neutron/dhcp_agent.ini file

```ini
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
[agent]
[ovs]
```

* Edit the /etc/neutron/metadata_agent.ini file

```ini
[DEFAULT]
nova_metadata_ip = controller0
metadata_proxy_shared_secret = poc#pass
[agent]
[cache]
```

* Create the symbolic link to /etc/neutron/plugin.ini

```bash
# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

* Initilize the neutron database

```bash
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
                --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

* Enabling and starting the services

```bash
# systemctl enable neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent neutron-l3-agent
# systemctl start neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent neutron-l3-agent
```

#### Step3: Installing and configuring the Neutron (Networking service) on Compute nodes
The following tasks should be done on the compute1 and compute2 nodes.

* 
