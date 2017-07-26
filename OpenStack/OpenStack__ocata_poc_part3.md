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
                             --provider-physical-network external_network --external provider_network
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2017-07-26T07:17:02Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 625e5d2c-2f03-4bd5-940c-72242ba69327 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| mtu                       | 1500                                 |
| name                      | provider_network                     |
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
| updated_at                | 2017-07-26T07:17:02Z                 |
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
| created_at                | 2017-07-26T07:29:33Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | ddbabaaa-169f-4ff6-8be5-630f75d1bee7 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| mtu                       | 1450                                 |
| name                      | tenant_network                       |
| port_security_enabled     | True                                 |
| project_id                | 6a15ffaacaad4c84b78d72d740229a7b     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 95                                   |
| qos_policy_id             | None                                 |
| revision_number           | 3                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| updated_at                | 2017-07-26T07:29:33Z                 |
+---------------------------+--------------------------------------+
```

* Verifying the networks just created

```bash
$ source ~/admin-openrc
$ openstack network list
+--------------------------------------+------------------+---------+
| ID                                   | Name             | Subnets |
+--------------------------------------+------------------+---------+
| 625e5d2c-2f03-4bd5-940c-72242ba69327 | provider_network |         |
| ddbabaaa-169f-4ff6-8be5-630f75d1bee7 | tenant_network   |         |
+--------------------------------------+------------------+---------+
```

#### Step2: Creating subnets with openstack CLI

* Create the subnet for provider_network

```bash
$ source ~/admin-openrc
$ openstack subnet create --project poc --project-domain default --subnet-range 172.16.9.0/24 \
                          --gateway 172.16.9.1 --network provider_network \
                          --allocation-pool start=172.16.9.160,end=172.16.9.169 --dns-nameserver 8.8.8.8 provider_network_subnet
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 172.16.9.160-172.16.9.169            |
| cidr              | 172.16.9.0/24                        |
| created_at        | 2017-07-26T07:39:43Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.8.8                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 172.16.9.1                           |
| host_routes       |                                      |
| id                | 0086f62f-8aa6-4666-ac7b-a62d1f2286e7 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | provider_network_subnet              |
| network_id        | 625e5d2c-2f03-4bd5-940c-72242ba69327 |
| project_id        | 6a15ffaacaad4c84b78d72d740229a7b     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2017-07-26T07:39:43Z                 |
+-------------------+--------------------------------------+
```

* Create the subnet for tenant_network

```bash
$ source ~/admin-openrc
$ openstack subnet create --project poc --project-domain default --subnet-range 192.168.154.0/24 \
                          --network tenant_network --allocation-pool start=192.168.154.2,end=192.168.154.254 \
                          --dns-nameserver 8.8.8.8 tenant_network_subnet
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.154.2-192.168.154.254        |
| cidr              | 192.168.154.0/24                     |
| created_at        | 2017-07-26T07:48:20Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.8.8                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.154.1                        |
| host_routes       |                                      |
| id                | 5453133f-b2a9-4900-9e1a-fbac90ef139f |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | tenant_network_subnet                |
| network_id        | ddbabaaa-169f-4ff6-8be5-630f75d1bee7 |
| project_id        | 6a15ffaacaad4c84b78d72d740229a7b     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2017-07-26T07:48:20Z                 |
+-------------------+--------------------------------------+
```

* Verifying the subnets just created

```bash
$ source ~/admin-openrc
$ openstack subnet list
+--------------------------------------+-------------------------+--------------------------------------+------------------+
| ID                                   | Name                    | Network                              | Subnet           |
+--------------------------------------+-------------------------+--------------------------------------+------------------+
| 0086f62f-8aa6-4666-ac7b-a62d1f2286e7 | provider_network_subnet | 625e5d2c-2f03-4bd5-940c-72242ba69327 | 172.16.9.0/24    |
| 5453133f-b2a9-4900-9e1a-fbac90ef139f | tenant_network_subnet   | ddbabaaa-169f-4ff6-8be5-630f75d1bee7 | 192.168.154.0/24 |
+--------------------------------------+-------------------------+--------------------------------------+------------------+
```

#### Step3: Creating the router and adding subnets with openstack CLI

* Create the router

```bash
$ source ~/admin-openrc
$ openstack router create --project poc --project-domain default router
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2017-07-26T08:28:12Z                 |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   | None                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | 20e99e41-c85c-4e76-a6bc-5d89ca7a2840 |
| name                    | router                               |
| project_id              | 6a15ffaacaad4c84b78d72d740229a7b     |
| revision_number         | None                                 |
| routes                  |                                      |
| status                  | ACTIVE                               |
| updated_at              | 2017-07-26T08:28:12Z                 |
+-------------------------+--------------------------------------+
```

* Adding the both subnets to router just created

```bash
$ source ~/admin-openrc
$ openstack router set --external-gateway provider_network router
$ openstack router add subnet router tenant_network_subnet

-- recheck router metadata
$ openstack router show router
+-------------------------+-----------------------------------------------------------+
| Field                   | Value                                                     |
+-------------------------+-----------------------------------------------------------+
| admin_state_up          | UP                                                        |
| availability_zone_hints |                                                           |
| availability_zones      | nova                                                      |
| created_at              | 2017-07-26T08:28:12Z                                      |
| description             |                                                           |
| distributed             | False                                                     |
| external_gateway_info   | {"network_id": "625e5d2c-2f03-4bd5-940c-72242ba69327",    |
|                         | "enable_snat": true, "external_fixed_ips": [{"subnet_id": |
|                         | "0086f62f-8aa6-4666-ac7b-a62d1f2286e7", "ip_address":     |
|                         | "172.16.9.163"}]}                                         |
| flavor_id               | None                                                      |
| ha                      | False                                                     |
| id                      | 20e99e41-c85c-4e76-a6bc-5d89ca7a2840                      |
| name                    | router                                                    |
| project_id              | 6a15ffaacaad4c84b78d72d740229a7b                          |
| revision_number         | 15                                                        |
| routes                  |                                                           |
| status                  | ACTIVE                                                    |
| updated_at              | 2017-07-26T08:59:12Z                                      |
+-------------------------+-----------------------------------------------------------+
```

* Verifying the connectivity
:star:This check tasks are conducted from network1 node.

```bash
# ip netns
qrouter-20e99e41-c85c-4e76-a6bc-5d89ca7a2840 (id: 2)
qdhcp-ddbabaaa-169f-4ff6-8be5-630f75d1bee7 (id: 1)
qdhcp-625e5d2c-2f03-4bd5-940c-72242ba69327 (id: 0)

# ip netns exec qdhcp-625e5d2c-2f03-4bd5-940c-72242ba69327 ip addr show
...snip...
2: ns-bd98b617-9e@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP qlen 1000
    link/ether fa:16:3e:de:25:31 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.9.160/24 brd 172.16.9.255 scope global ns-bd98b617-9e
       valid_lft forever preferred_lft forever
    inet 169.254.169.254/16 brd 169.254.255.255 scope global ns-bd98b617-9e
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fede:2531/64 scope link
       valid_lft forever preferred_lft forever
       
# ip netns exec qdhcp-625e5d2c-2f03-4bd5-940c-72242ba69327 ip route
default via 172.16.9.1 dev ns-bd98b617-9e
169.254.0.0/16 dev ns-bd98b617-9e  proto kernel  scope link  src 169.254.169.254
172.16.9.0/24 dev ns-bd98b617-9e  proto kernel  scope link  src 172.16.9.160

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
$ openstack security group create --project poc --project-domain default poc_securitygroup
+-----------------+------------------------------------------------------------------------+
| Field           | Value                                                                  |
+-----------------+------------------------------------------------------------------------+
| created_at      | 2017-07-26T10:29:03Z                                                   |
| description     | poc_securitygroup                                                      |
| id              | c5229e9e-2922-4ea0-9aa3-6ee882ca5436                                   |
| name            | poc_securitygroup                                                      |
| project_id      | 6a15ffaacaad4c84b78d72d740229a7b                                       |
| revision_number | 1                                                                      |
| rules           | created_at='2017-07-26T10:29:03Z', direction='egress',                 |
|                 | ethertype='IPv4', id='85355dee-8086-4972-b0d3-c0a900846dd4',           |
|                 | revision_number='1', updated_at='2017-07-26T10:29:03Z'                 |
|                 | created_at='2017-07-26T10:29:03Z', direction='egress',                 |
|                 | ethertype='IPv6', id='2ad36168-c700-4945-a3f8-e7b2c9b707a5',           |
|                 | revision_number='1', updated_at='2017-07-26T10:29:03Z'                 |
| updated_at      | 2017-07-26T10:29:03Z                                                   |
+-----------------+------------------------------------------------------------------------+
```

* Adding rules to just created security group

```bash
$ source ~/admin-openrc
$ openstack security group rule create --protocol icmp poc_securitygroup
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-07-26T10:32:41Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 12c8c3ae-d770-4e2c-bc1b-b5937573139a |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 6f173e1f310146c9883a2c50d1336bc0     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | c5229e9e-2922-4ea0-9aa3-6ee882ca5436 |
| updated_at        | 2017-07-26T10:32:41Z                 |
+-------------------+--------------------------------------+
$ openstack security group rule create --protocol tcp --dst-port 22 poc_securitygroup
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-07-26T10:44:30Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 999df9ea-d592-415b-ae6f-d3fdcdb2e573 |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 6f173e1f310146c9883a2c50d1336bc0     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | c5229e9e-2922-4ea0-9aa3-6ee882ca5436 |
| updated_at        | 2017-07-26T10:44:30Z                 |
+-------------------+--------------------------------------+
```

#### Step5: Creating the flavor with openstack CLI
