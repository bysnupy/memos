## Staging environment installation memo

### Introduction
This tutorial is simple documents of staging environment deployment for providing some services.

### Environment Information

* OpenStack version: Ocata
* Host OS: CentOS 7.3
* Deployment method: RDO and packstack
* Server specifications (physical servers)

Node | CPU | Memory | NICs | Storages
ctrl1.host.local| 4 Cores, Xeon | 8 GiB | 2 NICs | 200 GB, SATA, no RAID
com1.host.local | 12 Cores (HT: 24), Xeon | 64 GiB |6 NICs | 1 TB, SAS, 10 RAID 

