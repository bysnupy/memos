## PoC environment deployment memo
### 1. Environment information
:star: The nodes are virtual machines based KVM.
* OpenStack version: Ocata
* OS: CentOS 7.3
* Deployment method: RDO
* Server Specifications (total: 5 nodes):
  - controller x1 : 4x cores, 4 GiB, 1x NIC 1Gbps, 50 GB
  - compute x2    : 4x cores, 3x NIC 1Gbps, 4 GiB, 50 GB
  - network x1    : 2 cores, 3x NIC 1Gbps, 4 GiB, 30 GB
  - block x1      : 2 cores, 2x NIC 1Gbps, 4 GiB, 30 GB (+secondary volume 30 GB)
 
### 2. Logical nodes diagram
