## PoC environment construction tutorial part3

### Introduction
It is the tutorial about installing openstack for PoC environment.<br/>
The part3 is describing mainly network configurations through CLI.

<[The Part1 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part1.md)><br/>
<[The Part2 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part2.md)>

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

The following commands can be executed from any node which installed python-openstackclient package if you have admin credentials.
But this time the following commands were executed from controllor0 node unless otherwise noted.

* Each network attributes table

Network Name|Project|Network Type|Gateway|External|DHCP
------------|-------|------------|-------|--------|----
provider_network1|poc|flat|Yes|Yes|No
tenant_network1|poc|vxlan|No|No|Yes

#### Step1: Creating External(Provider) and Private local(Tenant) networks with openstack CLI

* Create the external network which names provider_network

```bash
$ source ~/admin-openrc
$ openstack network create --project poc --project-domain default --provider-network-type flat \
                             --provider-physical-network external_network --external provider_network1
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2017-07-27T08:31:42Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | ab964185-f3f6-4915-a388-fb099273d2da |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| mtu                       | 1500                                 |
| name                      | provider_network1                    |
| port_security_enabled     | True                                 |
| project_id                | 6a15ffaacaad4c84b78d72d740229a7b     |
| provider:network_type     | flat                                 |
| provider:physical_network | external_network                     |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 4                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| updated_at                | 2017-07-27T08:31:43Z                 |
+---------------------------+--------------------------------------+
```

* Create the private local network which names tenant_network

```bash
$ source ~/admin-openrc
$ openstack network create --project poc --project-domain default tenant_network
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2017-07-27T08:32:24Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 1877fce3-06af-4d40-9b88-ae2b9fd2ee83 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| mtu                       | 1450                                 |
| name                      | tenant_network1                      |
| port_security_enabled     | True                                 |
| project_id                | 6a15ffaacaad4c84b78d72d740229a7b     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 50                                   |
| qos_policy_id             | None                                 |
| revision_number           | 3                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| updated_at                | 2017-07-27T08:32:24Z                 |
+---------------------------+--------------------------------------+
```

* Verifying the networks just created

```bash
$ source ~/admin-openrc
$ openstack network list
+--------------------------------------+-------------------+---------+
| ID                                   | Name              | Subnets |
+--------------------------------------+-------------------+---------+
| 1877fce3-06af-4d40-9b88-ae2b9fd2ee83 | tenant_network1   |         |
| ab964185-f3f6-4915-a388-fb099273d2da | provider_network1 |         |
+--------------------------------------+-------------------+---------+
```

#### Step2: Creating subnets with openstack CLI

* Create the subnet for provider_network1

```bash
$ source ~/admin-openrc
$ openstack subnet create --project poc --project-domain default --subnet-range 172.16.9.0/24 \
                          --gateway 172.16.9.1 --network provider_network1 --no-dhcp \
                          --allocation-pool start=172.16.9.160,end=172.16.9.169 --dns-nameserver 8.8.8.8 provider_network1_subnet
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 172.16.9.160-172.16.9.169            |
| cidr              | 172.16.9.0/24                        |
| created_at        | 2017-07-27T08:34:53Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.8.8                              |
| enable_dhcp       | False                                |
| gateway_ip        | 172.16.9.1                           |
| host_routes       |                                      |
| id                | 207a819f-ddc3-461a-91c6-6d46445b11c1 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | provider_network1_subnet             |
| network_id        | ab964185-f3f6-4915-a388-fb099273d2da |
| project_id        | 6a15ffaacaad4c84b78d72d740229a7b     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2017-07-27T08:34:53Z                 |
+-------------------+--------------------------------------+

```

* Create the subnet for tenant_network1

```bash
$ source ~/admin-openrc
$ openstack subnet create --project poc --project-domain default --subnet-range 192.168.155.0/24 \
                          --network tenant_network1 --allocation-pool start=192.168.154.2,end=192.168.155.254 \
                          --dns-nameserver 8.8.8.8 tenant_network1_subnet
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.155.2-192.168.155.254        |
| cidr              | 192.168.155.0/24                     |
| created_at        | 2017-07-27T08:37:22Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.8.8                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.155.1                        |
| host_routes       |                                      |
| id                | 56df2a6d-246f-46a3-bd23-ca42f975f022 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | tenant_network1_subnet               |
| network_id        | 1877fce3-06af-4d40-9b88-ae2b9fd2ee83 |
| project_id        | 6a15ffaacaad4c84b78d72d740229a7b     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2017-07-27T08:37:22Z                 |
+-------------------+--------------------------------------+
```

* Verifying the subnets just created

```bash
$ source ~/admin-openrc
$ openstack subnet list
+--------------------------------------+--------------------------+--------------------------------------+------------------+
| ID                                   | Name                     | Network                              | Subnet           |
+--------------------------------------+--------------------------+--------------------------------------+------------------+
| 207a819f-ddc3-461a-91c6-6d46445b11c1 | provider_network1_subnet | ab964185-f3f6-4915-a388-fb099273d2da | 172.16.9.0/24    |
| 56df2a6d-246f-46a3-bd23-ca42f975f022 | tenant_network1_subnet   | 1877fce3-06af-4d40-9b88-ae2b9fd2ee83 | 192.168.155.0/24 |
+--------------------------------------+--------------------------+--------------------------------------+------------------+
```

#### Step3: Creating the router and adding subnets with openstack CLI

* Create the router1

```bash
$ source ~/admin-openrc
$ openstack router create --project poc --project-domain default router1
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2017-07-27T08:39:01Z                 |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   | None                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | e6d3d85d-10a9-4477-8876-6fb11909d21d |
| name                    | router1                              |
| project_id              | 6a15ffaacaad4c84b78d72d740229a7b     |
| revision_number         | None                                 |
| routes                  |                                      |
| status                  | ACTIVE                               |
| updated_at              | 2017-07-27T08:39:01Z                 |
+-------------------------+--------------------------------------+
```

* Adding the both subnets to the router1 just created

```bash
$ source ~/admin-openrc
$ openstack router set --external-gateway provider_network1 router1
$ openstack router add subnet router1 tenant_network1_subnet

-- recheck router metadata
$ openstack router show router1
+-------------------------+----------------------------------------------------------------------------------------------------+
| Field                   | Value                                                                                              |
+-------------------------+----------------------------------------------------------------------------------------------------+
| admin_state_up          | UP                                                                                                 |
| availability_zone_hints |                                                                                                    |
| availability_zones      |                                                                                                    |
| created_at              | 2017-07-27T08:39:01Z                                                                               |
| description             |                                                                                                    |
| distributed             | False                                                                                              |
| external_gateway_info   | {"network_id": "ab964185-f3f6-4915-a388-fb099273d2da", "enable_snat": true, "external_fixed_ips":  |
|                         | [{"subnet_id": "207a819f-ddc3-461a-91c6-6d46445b11c1", "ip_address": "172.16.9.160"}]}             |
| flavor_id               | None                                                                                               |
| ha                      | False                                                                                              |
| id                      | e6d3d85d-10a9-4477-8876-6fb11909d21d                                                               |
| name                    | router1                                                                                            |
| project_id              | 6a15ffaacaad4c84b78d72d740229a7b                                                                   |
| revision_number         | 8                                                                                                  |
| routes                  |                                                                                                    |
| status                  | ACTIVE                                                                                             |
| updated_at              | 2017-07-27T08:40:27Z                                                                               |
+-------------------------+----------------------------------------------------------------------------------------------------+
```

* Verifying the connectivity

This network topology image is from dashboard.

![Network Topology1](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_network1.png)

:warning:You must check the hypervisor network security settings if you install to VMs on the hypervisor.<br/>
It result in the connectivity trouble to external network, as  the host servers had dropped the packets from the VMs

The vSwitch security settings need to change as follows on the ESXi

![Warning ESXi](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_esxi1.png)

The vNIC Profiles settings need to change as follows on the oVirt or RHEV

![Warning oVirt](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_ovirt1.png)

:star:This check commands were executed from network1 node.

```bash
-- identify the network namespaces
# ip netns
qrouter-e6d3d85d-10a9-4477-8876-6fb11909d21d (id: 1)
qdhcp-1877fce3-06af-4d40-9b88-ae2b9fd2ee83 (id: 0)

-- check the tenant_network1 interface (qr-XXX) and provider_network1 interface (qg-XXX) 
# ip netns exec qrouter-e6d3d85d-10a9-4477-8876-6fb11909d21d ip address show
...snip...
2: qr-877068db-6e@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP qlen 1000
    link/ether fa:16:3e:8b:c8:c5 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.155.1/24 brd 192.168.155.255 scope global qr-877068db-6e
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe8b:c8c5/64 scope link
       valid_lft forever preferred_lft forever
3: qg-a7a88215-3e@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether fa:16:3e:41:6b:6a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.9.160/24 brd 172.16.255.255 scope global qg-a7a88215-3e
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe41:6b6a/64 scope link
       valid_lft forever preferred_lft forever

-- show the routing table
# ip netns exec qrouter-e6d3d85d-10a9-4477-8876-6fb11909d21d ip route list
default via 172.16.9.1 dev qg-a7a88215-3e
172.16.9.0/24 dev qg-a7a88215-3e  proto kernel  scope link  src 172.16.9.160
192.168.155.0/24 dev qr-877068db-6e  proto kernel  scope link  src 192.168.155.1

-- ping the external gateway
# ip netns exec qrouter-e6d3d85d-10a9-4477-8876-6fb11909d21d ping -c3 172.16.9.1
PING 172.16.9.1 (172.16.9.1) 56(84) bytes of data.
64 bytes from 172.16.9.1: icmp_seq=1 ttl=255 time=2.30 ms
64 bytes from 172.16.9.1: icmp_seq=2 ttl=255 time=2.42 ms
64 bytes from 172.16.9.1: icmp_seq=3 ttl=255 time=2.50 ms

--- 172.16.9.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 2.307/2.411/2.506/0.081 ms

```

#### Step4: Creating the securitygroup with openstack CLI

* Create new security group poc_securitygroup

```bash
$ source ~/admin-openrc
$ openstack security group create --project poc --project-domain default poc_secgroup
+-----------------+------------------------------------------------------------------------+
| Field           | Value                                                                  |
+-----------------+------------------------------------------------------------------------+
| created_at      | 2017-07-27T09:08:36Z                                                   |
| description     | poc_secgroup                                                           |
| id              | fb398d68-07a9-4241-9185-8521ec27c973                                   |
| name            | poc_secgroup                                                           |
| project_id      | 6a15ffaacaad4c84b78d72d740229a7b                                       |
| revision_number | 1                                                                      |
| rules           | created_at='2017-07-27T09:08:36Z', direction='egress',                 |
|                 | ethertype='IPv4', id='06f34a3d-9aa7-44e7-a3b6-b63dc6bfb408',           |
|                 | revision_number='1', updated_at='2017-07-27T09:08:36Z'                 |
|                 | created_at='2017-07-27T09:08:36Z', direction='egress',                 |
|                 | ethertype='IPv6', id='20cc80dd-60f5-4d53-bcb2-5205d879e6cd',           |
|                 | revision_number='1', updated_at='2017-07-27T09:08:36Z'                 |
| updated_at      | 2017-07-27T09:08:36Z                                                   |
+-----------------+------------------------------------------------------------------------+
```

* Adding rules to just created security group

```bash
$ source ~/admin-openrc

-- allow ssh access from external
$ openstack security group rule create --protocol tcp --dst-port 22 poc_secgroup
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-07-27T09:09:43Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 2809512f-11fc-4746-b980-d3e1a7b37cc2 |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 6f173e1f310146c9883a2c50d1336bc0     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | fb398d68-07a9-4241-9185-8521ec27c973 |
| updated_at        | 2017-07-27T09:09:43Z                 |
+-------------------+--------------------------------------+

-- allow all icmp from external
$ openstack security group rule create --protocol icmp poc_secgroup
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-07-27T09:14:33Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | d94e5d0d-2420-4152-bba5-68fa4f877d6e |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 6f173e1f310146c9883a2c50d1336bc0     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | fb398d68-07a9-4241-9185-8521ec27c973 |
| updated_at        | 2017-07-27T09:14:33Z                 |
+-------------------+--------------------------------------+
```

* check the security group rules just added

```bash
$ source ~/admin-openrc
$ openstack security group rule list poc_secgroup
+-----------------------+-------------+-----------+------------+-----------------------+
| ID                    | IP Protocol | IP Range  | Port Range | Remote Security Group |
+-----------------------+-------------+-----------+------------+-----------------------+
| 06f34a3d-9aa7-44e7-a3 | None        | None      |            | None                  |
| b6-b63dc6bfb408       |             |           |            |                       |
| 20cc80dd-60f5-4d53-bc | None        | None      |            | None                  |
| b2-5205d879e6cd       |             |           |            |                       |
| 2809512f-11fc-4746-b9 | tcp         | 0.0.0.0/0 | 22:22      | None                  |
| 80-d3e1a7b37cc2       |             |           |            |                       |
| d94e5d0d-2420-4152-bb | icmp        | 0.0.0.0/0 |            | None                  |
| a5-68fa4f877d6e       |             |           |            |                       |
+-----------------------+-------------+-----------+------------+-----------------------+
```

#### Step5: Creating the floating IP addresses and flavors with openstack CLI

* Get the floating IP

In this case I was gotten the 172.16.9.162 floating IP.

```bash
$ source ~/admin-openrc
$ openstack floating ip create provider_network1
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2017-07-28T03:43:20Z                 |
| description         |                                      |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.16.9.165                         |
| floating_network_id | ab964185-f3f6-4915-a388-fb099273d2da |
| id                  | 5621480c-ec5f-4d5b-8648-a3cb68adb9fa |
| name                | None                                 |
| port_id             | None                                 |
| project_id          | 6a15ffaacaad4c84b78d72d740229a7b     |
| revision_number     | 1                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| updated_at          | 2017-07-28T03:43:20Z                 |
+---------------------+--------------------------------------+

$ openstack floating ip list
+--------------------+---------------------+------------------+------+--------------------+---------------------+
| ID                 | Floating IP Address | Fixed IP Address | Port | Floating Network   | Project             |
+--------------------+---------------------+------------------+------+--------------------+---------------------+
| 5621480c-ec5f-4d5b | 172.16.9.165        | None             | None | ab964185-f3f6-4915 | 6a15ffaacaad4c84b78 |
| -8648-a3cb68adb9fa |                     |                  |      | -a388-fb099273d2da | d72d740229a7b       |
+--------------------+---------------------+------------------+------+--------------------+---------------------+
```

* Create the flavor entity

```bash
$ openstack flavor create --project poc --project-domain default \
                          --private --vcpus 2 --ram 1024 --disk 5 --swap 512 p2.small
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| disk                       | 5                                    |
| id                         | 34be659e-a4ba-4b35-aa2d-fc9799e2d412 |
| name                       | p2.small                             |
| os-flavor-access:is_public | False                                |
| properties                 |                                      |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       | 512                                  |
| vcpus                      | 2                                    |
+----------------------------+--------------------------------------+

$ openstack flavor list --private
+-----------------------------+----------+------+------+-----------+-------+-----------+
| ID                          | Name     |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+-----------------------------+----------+------+------+-----------+-------+-----------+
| 34be659e-a4ba-4b35-aa2d-    | p2.small | 1024 |    5 |         0 |     2 | False     |
| fc9799e2d412                |          |      |      |           |       |           |
+-----------------------------+----------+------+------+-----------+-------+-----------+
```

#### Step6: Creating the keypairs and instance with openstack CLI

* Generating the keypair

```bash
$ source ~/admin-openrc

-- create the new keypair and save the private key as poc_keypair1.key
$ openstack keypair create poc_keypair1 | tee poc_keypair1.key
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAupJPHgelKVK2px453FyicG4ZzuU7akXJcIzk2nqhCSNmAguI
2/Zt/JUZA15LQU+sQja7D/NL13KIDIX+YPAuTEfmE1oeE3Wanfex+nT4ZRKZtwVS
EDkbVNaYlk/QPmcXwimYMwum1TmhUhwSJyH42aw/5rN0z8vYkB5RHBvHFy9IToMa
Re7kytgJyFdJpeJK5tI9dWgt8j+GyN+Mk1Xyv6XAiGv+EewYdzsMKMoJ1Db666Pz
...snip..
OGd6gWm0aOWiJ2puv8aSXpEvwCjq3lY7592uixLVQaLfOR6MFPmFBnbAtpV1auzE
QmroZP+hst4fMkIGiGh95nnHwU4XsueB5hp/Q+xeBq6S7oCDu2WohwC/Ll2DCFl2
ynI5AoGAe73FU7CJM5ExgHLlMwxi6PLU2ZjTi9W4wSH/zWpJH/Lbsm0G2O3qFALs
aDxuW3H5LBPbxN0Rx5P6x5jyJgBAfoy9tB3cyQoMAI8xXulKUA1YWEMkLna916+8
GBU4dfKUKBcommTfRWXCZG2m4rcYlqHLL/ITayDK6tqm/hVfNzQ=
-----END RSA PRIVATE KEY-----

-- change access mode 0600 or 0400 to protect from the warning when you use this key to access remote VMs.
$ chmod 0400 poc_keypair1.key

$ openstack keypair list
+--------------+-------------------------------------------------+
| Name         | Fingerprint                                     |
+--------------+-------------------------------------------------+
| poc_keypair1 | 30:4c:d9:26:a3:8f:df:f9:84:ad:22:c3:77:cc:10:24 |
+--------------+-------------------------------------------------+
```

* Launch the instance

:star:The following commands were executed by pocuser

```bash
$ source ~/pocuser-openrc
$ openstack server create  --flavor 34be659e-a4ba-4b35-aa2d-fc9799e2d412 --image cirros   --nic net-id=1877fce3-06af-4d40-9b88-ae2b9fd2ee83 --security-group poc_secgroup --key-name poc_keypair1 pocvm1
+-----------------------------+-------------------------------------------------+
| Field                       | Value                                           |
+-----------------------------+-------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                          |
| OS-EXT-AZ:availability_zone |                                                 |
| OS-EXT-STS:power_state      | NOSTATE                                         |
| OS-EXT-STS:task_state       | scheduling                                      |
| OS-EXT-STS:vm_state         | building                                        |
| OS-SRV-USG:launched_at      | None                                            |
| OS-SRV-USG:terminated_at    | None                                            |
| accessIPv4                  |                                                 |
| accessIPv6                  |                                                 |
| addresses                   |                                                 |
| adminPass                   | DPebJU3AU4if                                    |
| config_drive                |                                                 |
| created                     | 2017-07-28T06:33:32Z                            |
| flavor                      | p2.small (34be659e-a4ba-4b35-aa2d-fc9799e2d412) |
| hostId                      |                                                 |
| id                          | a55569d8-4175-4b5b-9310-aa4978e29ce5            |
| image                       | cirros (3dd7948f-6c14-4aaf-802d-be2716a1ad4a)   |
| key_name                    | poc_keypair1                                    |
| name                        | pocvm1                                          |
| progress                    | 0                                               |
| project_id                  | 6a15ffaacaad4c84b78d72d740229a7b                |
| properties                  |                                                 |
| security_groups             | name='poc_secgroup'                             |
| status                      | BUILD                                           |
| updated                     | 2017-07-28T06:33:32Z                            |
| user_id                     | be75a0f788104233bf2bd9c6f5e37ec7                |
| volumes_attached            |                                                 |
+-----------------------------+-------------------------------------------------+

-- check the instance state is running
$ openstack server show pocvm1
+-----------------------------+----------------------------------------------------------+
| Field                       | Value                                                    |
+-----------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                   |
| OS-EXT-AZ:availability_zone | nova                                                     |
| OS-EXT-STS:power_state      | Running                                                  |
...snip...
| properties                  |                                                          |
| security_groups             | name='poc_secgroup'                                      |
| status                      | ACTIVE                                                   |
| updated                     | 2017-07-28T06:33:52Z                                     |
| user_id                     | be75a0f788104233bf2bd9c6f5e37ec7                         |
| volumes_attached            |                                                          |
+-----------------------------+----------------------------------------------------------+
```

* Allocating the floating IP to the instance

```bash
$ source ~/pocuser-openrc
$ openstack server add floating ip pocvm1 172.16.9.165
$ openstack server show pocvm1
+-----------------------------+----------------------------------------------------------+
| Field                       | Value                                                    |
+-----------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                   |
| OS-EXT-AZ:availability_zone | nova                                                     |
| OS-EXT-STS:power_state      | Running                                                  |
| OS-EXT-STS:task_state       | None                                                     |
| OS-EXT-STS:vm_state         | active                                                   |
| OS-SRV-USG:launched_at      | 2017-07-28T06:33:51.000000                               |
| OS-SRV-USG:terminated_at    | None                                                     |
| accessIPv4                  |                                                          |
| accessIPv6                  |                                                          |
| addresses                   | tenant_network1=192.168.155.11, 172.16.9.165             |
...snip...
| security_groups             | name='poc_secgroup'                                      |
| status                      | ACTIVE                                                   |
| updated                     | 2017-07-28T06:33:52Z                                     |
| user_id                     | be75a0f788104233bf2bd9c6f5e37ec7                         |
| volumes_attached            |                                                          |
+-----------------------------+----------------------------------------------------------+
```

#### Step7: Verifying accessibility to new instance just created through SSH.

:star:Where you execute the tasks on does not matter while on the same segment with external network.
But I had executed  the following ones on the controller0 node.

* check the accessibility through CLI

```bash
-- ping
$ ping -c3 172.16.9.165
PING 172.16.9.165 (172.16.9.165) 56(84) bytes of data.
64 bytes from 172.16.9.165: icmp_seq=1 ttl=63 time=3.50 ms
64 bytes from 172.16.9.165: icmp_seq=2 ttl=63 time=1.07 ms
64 bytes from 172.16.9.165: icmp_seq=3 ttl=63 time=1.24 ms

--- 172.16.9.165 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 1.077/1.943/3.509/1.109 ms

-- ssh with private key
$ ssh -i poc_keypair1.key cirros@172.16.9.165
The authenticity of host '172.16.9.165 (172.16.9.165)' can't be established.
RSA key fingerprint is 00:4c:96:1c:a3:a0:aa:8d:6b:e5:db:92:45:ec:b2:03.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.9.165' (RSA) to the list of known hosts.

$ hostname
pocvm1

$ ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:61:36:77 brd ff:ff:ff:ff:ff:ff
    inet 192.168.155.11/24 brd 192.168.155.255 scope global eth0
    inet6 fe80::f816:3eff:fe61:3677/64 scope link
       valid_lft forever preferred_lft forever
       
$ ping -c3 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=56 time=3.902 ms
64 bytes from 8.8.8.8: seq=1 ttl=56 time=3.109 ms
64 bytes from 8.8.8.8: seq=2 ttl=56 time=3.063 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 3.063/3.358/3.902 ms
$
```

* check the vnc console through Horizon

Network Topology is here.

![Network Topology2](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_network2.png)

* Access the vnc pages

Move the 'Instances' pages as 'Poject' -> 'Compute' -> 'Instacnes' as follows

![VNC1](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_vnc1.png)

Show the VNC console as clicking the vm name link -> 'Console' as follows

![VNC2](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_vnc2.png)

:warning:If you encountered the instance hang up at booting up, first you check the libvirt log on the compute node running the instances.

```bash
# cat /var/log/libvirt/qemu/instance-00000001.log | grep -i ^warning
warning: TCG doesn't support requested feature: CPUID.01H:EDX.vme [bit 1]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.fma [bit 12]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.pcid [bit 17]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.x2apic [bit 21]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.tsc-deadline [bit 24]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.osxsave [bit 27]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.avx [bit 28]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.f16c [bit 29]
warning: TCG doesn't support requested feature: CPUID.01H:ECX.rdrand [bit 30]
warning: TCG doesn't support requested feature: CPUID.07H:EBX.hle [bit 4]
warning: TCG doesn't support requested feature: CPUID.07H:EBX.avx2 [bit 5]
warning: TCG doesn't support requested feature: CPUID.07H:EBX.erms [bit 9]
...snip...
```

And check the VNC console through Horizon.
If you would find the messages like 'Starting up ...', you need to modify the /etc/nova/nova.conf file as follows

```ini
...snip...
[libvirt]
virt_type = qemu
cpu_mode=none
[matchmaker_redis]
[metrics]
..snip...
```

In my case it can be solved by editing like 'cpu_mode=none'.

