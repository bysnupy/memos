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

#### Test data preparation

* CPU

CPU|VMware ESXi|KVM (OpenStack Nova Compute)
-|-|-
Min|57727|55735|
Avg|58178|57677|
Max|58564|58664|
Dev|280  |1003 |

* Memory

Memory|VMware ESXi|KVM (OpenStack Nova Compute)
-|-|-
Min|31878949|22531790|
Avg|33041855|24340954|
Max|33648671|26051382|
Dev|489027  |1083923 |

* FileIO

Random write|VMware ESXi|KVM (OpenStack Nova Compute)
-|-|-
Min|51651|39428|
Avg|59156|41598|
Max|64938|43885|
Dev|4179 |1275 |

Random read|VMware ESXi|KVM (OpenStack Nova Compute)
-|-|-
Min|2383637|2295254|
Avg|2459549|2691738|
Max|2496902|2835669|
Dev|41415  |187250 |

### Conclusion
