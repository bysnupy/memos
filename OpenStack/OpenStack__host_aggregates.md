## Host Aggregates

#### Introduction

OpenStack have Availability Zone and Host Aggregates mechanisms to provide instances on the specific hosts.

Mechnism | Usual use cases | End user visible
-|-|-
Availability Zone | Abstraction groups to be separated geographically for high availability | Yes, but you can only show the zone name, not hostname
Host Aggregates | Logical infrastructure resource groups for enhanced performance and specific administrations | No, just Administrator can control

The following steps describe the configurations for enabling Host aggregates.

#### Configuration Steps

This configurations are conducted on the node which are providing nova-scheduler service.




