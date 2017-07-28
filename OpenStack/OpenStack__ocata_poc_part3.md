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

:star:This check commands were executed from network1 node.

```bash
# ip netns
qdhcp-1877fce3-06af-4d40-9b88-ae2b9fd2ee83 (id: 2)
qdhcp-ab964185-f3f6-4915-a388-fb099273d2da (id: 1)
qrouter-e6d3d85d-10a9-4477-8876-6fb11909d21d (id: 0)

# ip netns exec qdhcp-ab964185-f3f6-4915-a388-fb099273d2da ip addr show
...snip...
2: ns-9791c213-f7@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether fa:16:3e:9e:9e:eb brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 169.254.169.254/16 brd 169.254.255.255 scope global ns-9791c213-f7
       valid_lft forever preferred_lft forever
    inet 172.16.9.161/16 brd 172.16.9.255 scope global ns-9791c213-f7
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe9e:9eeb/64 scope link
       valid_lft forever preferred_lft forever
       
# ip netns exec qdhcp-ab964185-f3f6-4915-a388-fb099273d2da ip route
default via 172.16.9.1 dev ns-9791c213-f7
169.254.0.0/16 dev ns-9791c213-f7  proto kernel  scope link  src 169.254.169.254
172.16.9.0/24 dev ns-9791c213-f7  proto kernel  scope link  src 172.16.9.161

# ip netns exec qrouter-20e99e41-c85c-4e76-a6bc-5d89ca7a2840 ping -c3 172.16.9.1
PING 172.16.9.1 (172.16.9.1) 56(84) bytes of data.
64 bytes from 172.16.9.1: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 172.16.9.1: icmp_seq=2 ttl=64 time=0.037 ms
64 bytes from 172.16.9.1: icmp_seq=3 ttl=64 time=0.058 ms

--- 172.16.9.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.037/0.048/0.058/0.008 ms

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
| created_at          | 2017-07-27T09:21:31Z                 |
| description         |                                      |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.16.9.162                         |
| floating_network_id | ab964185-f3f6-4915-a388-fb099273d2da |
| id                  | 8e3d00a2-7cee-4c97-88fa-a99af1c1d166 |
| name                | None                                 |
| port_id             | None                                 |
| project_id          | 6f173e1f310146c9883a2c50d1336bc0     |
| revision_number     | 1                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| updated_at          | 2017-07-27T09:21:31Z                 |
+---------------------+--------------------------------------+

$ openstack floating ip list
+-------------------+---------------------+------------------+------+-------------------+-------------------+
| ID                | Floating IP Address | Fixed IP Address | Port | Floating Network  | Project           |
+-------------------+---------------------+------------------+------+-------------------+-------------------+
| 8e3d00a2-7cee-    | 172.16.9.162        | None             | None | ab964185-f3f6-491 | 6f173e1f310146c98 |
| 4c97-88fa-        |                     |                  |      | 5-a388-fb099273d2 | 83a2c50d1336bc0   |
| a99af1c1d166      |                     |                  |      | da                |                   |
+-------------------+---------------------+------------------+------+-------------------+-------------------+
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
$ openstack keypair create poc_keypair1
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

$ openstack keypair list
+--------------+-------------------------------------------------+
| Name         | Fingerprint                                     |
+--------------+-------------------------------------------------+
| poc_keypair1 | 30:4c:d9:26:a3:8f:df:f9:84:ad:22:c3:77:cc:10:24 |
+--------------+-------------------------------------------------+
```

