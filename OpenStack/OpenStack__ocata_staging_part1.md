## Staging environment installation memo

### Introduction
This tutorial is simple documents of staging environment deployment for providing some services.

### Environment Information

* OpenStack version: Ocata
* Host OS: CentOS 7.3
* Deployment method: RDO and packstack
* Server specifications (physical servers)

Node | CPU | Memory | NICs | Storages
-----|-----|--------|------|---------
ctrl1.host.local| 4 Cores, Xeon | 8 GiB | 2 NICs | 200 GB, SATA, no RAID
com1.host.local | 12 Cores (HT: 24), Xeon | 64 GiB |6 NICs (but 3 use here) | 1 TB, SAS, 10 RAID 

* Node interfaces and components information

Node|Components|Interfaces
----|----------|----------
ctrl1.host.local|Horizon<br/>Keystone<br/>Glance<br/>Management APIs<br/>MariaDB<br/>RabbitMQ<br/>Memcached | team0: 172.16.9.170
com1.host.local|Nova<br/>Neutron | eth0: 172.16.9.171<br/>eth1:192.168.33.171<br/>eth2: no IP

