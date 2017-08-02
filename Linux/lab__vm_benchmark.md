## Simple benchmark test for a VM with sysbench on the different hypervisors

### Introduction

This simple benchmark was for verifying the influence of performance about guest VM in a different each hypervisor.

#### Benchmark items and using tools

Packages

Package | Version
-|-
sysbench | 1.0.6

Benchmark items

Item | Evaluation | Options
-|-|-
CPU |  number of events/10s | default + threads=8
Memory | number of events/10s | default
DISK IO | number of events/10s about random read/write | default

#### Environment

Hypervisors

Pattern | Hypervisor
-|-
Before | ESXi 5.0.x
After | KVM (OpenStack Ocata)

Guest OS (All resources were reserved for its sizes on the hypervisors)

Item | Value
-|-
OS | CentOS 7.3.1611 (3.10.0-514.26.1.el7.x86_64)
CPU | 8 Cores
Memory | 8 GiB
HDD | 50 GB

#### Test data

* ESXi




