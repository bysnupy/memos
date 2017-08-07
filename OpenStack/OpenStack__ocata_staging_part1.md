## Staging environment installation memo

### Introduction
This tutorial is simple documents of staging environment deployment for providing some services.

### Test conditions

To test as follows, 

No. | Test Conditions | Environments
-|-|-
1 | Installation test for 1 controller and 1 compute nodes using packstack | virtual servers
2 | Adding 1 compute node to existing nodes with packstack | virtual servers
3 | DVR configuration on the existing 2 compute nodes | virtual servers
4 | Installation test for 1 controller and 1 compute nodes using packstack | physical servers

### Environment Information

* OpenStack version: Ocata
* Host OS: CentOS 7.3
* Deployment method: RDO and packstack
* Test Server specifications (virtual servers)

Node | CPU | Memory | NICs | Storages
-----|-----|--------|------|---------
ctrl1.host.local| 8 Cores | 8 GiB | 2 NICs | 50 GB
com1.host.local | 8 Cores | 8 GiB | 3 NICs | 50 GB
com2.host.local | 8 Cores | 8 GiB | 3 NICs | 50 GB

* Test Server specifications (physical servers)

Node | CPU | Memory | NICs | Storages
-----|-----|--------|------|---------
ctrl1.host.local| 4 Cores, Xeon | 8 GiB | 2 NICs | 200 GB, SATA, no RAID
com1.host.local | 12 Cores (HT: 24), Xeon | 64 GiB |6 NICs (but just 3 NICs use here) | 1 TB, SAS, 10 RAID 

* Node interfaces and components information (virtual servers)

Node|Components|Interfaces|Interface roles
----|----------|----------|---------------
ctrl1.host.local|Horizon<br/>Keystone<br/>Glance<br/>Management APIs<br/>MariaDB<br/>RabbitMQ<br/>Memcached | team0: 172.16.9.170 | team0: Management
com1.host.local|Nova<br/>Neutron | eth0: 172.16.9.171<br/>eth1:192.168.33.171<br/>eth2: no IP | eth0: Management<br/>eth1: VM ineternal network (overlay network, tenant network)<br/>eth2: External network (Provider network)
com2.host.local|Nova<br/>Neutron | eth0: 172.16.9.172<br/>eth1:192.168.33.172<br/>eth2: no IP | eth0: Management<br/>eth1: VM ineternal network (overlay network, tenant network)<br/>eth2: External network (Provider network)

