## Simple benchmark test for a VM on the different hypervisors.

### Introduction

This simple benchmark was for verifying the influence of performance about guest VM in a different each hypervisor.

#### Benchmark items and using tools

Packages

Package | Version
-|-
sysbench | 1.0.6
bonnie++ | 1.97.3

Benchmark items

Item | Tool | Criteria
-|-|-
CPU | sysbench | number of events/10s (default)
Memory | sysbench | number of events/10s (default)
DISK IO | bonnie++ | Sequential/Random block access/file manipulations (create/read/write)

#### Environment

Hypervisors

Pattern | Hypervisor
-|-
Before | ESXi 5.0.x
After | KVM (OpenStack Ocata)

Guest OS 

Item | Value
-|-
OS | CentOS 7.3.1611 (3.10.0-514.26.1.el7.x86_64)
CPU | 8 Cores
Memory | 8 GiB
HDD | 50 GB

