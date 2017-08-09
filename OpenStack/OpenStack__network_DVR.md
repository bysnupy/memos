## Distributed Virtual Router

### Introduction

Describing the DVR configuration and verifying packet flow.

### Environment Information

* Version and plugin

Item|Value
-|-
OpenStack version | Ocata
L2 | Open vSwitch

* Roles per nodes

Node | Role | Neutron service | DVR mode
-|-|-|-
ctrl1 | Controller | neutron server | N/A
com1 | Compute and Networking | l3, dhcp, metadata, openvswitch | dvr_snat
com2 | Compute and Networking | l3, dhcp, metadata, openvswitch | dvr

### Configuration Steps

* 
