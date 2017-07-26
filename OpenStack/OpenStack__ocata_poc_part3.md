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

#### Step1: Creating External(Provider) and Private local(Tenant) networks with openstack CLI

* Create the external network which names provider_network

```bash
$ source ~/admin-openrc
$ $ openstack network create --project poc --project-domain default --provider-network-type flat \
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

