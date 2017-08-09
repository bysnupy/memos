## Distributed Virtual Router

### Introduction

Describing the DVR configuration and verifying packet flow.

### Environment Information

* Version and plugin

Item|Value
-|-
OpenStack version | Ocata
L2 plugin | Open vSwitch
Tenant Network | VxLAN

* Roles per nodes

Node | Role | Neutron service | DVR mode
-|-|-|-
ctrl1 | Controller | neutron server | N/A
com1 | Compute and Networking | l3, dhcp, metadata, openvswitch | dvr_snat
com2 | Compute and Networking | l3, dhcp, metadata, openvswitch | dvr

### Configuration Steps

* Edit /etc/neutron/neutron.conf file on all nodes

```
[DEFAULT]
...snip...
router_distributed = True
...snip...
```

* Edit /etc/neutron/plugins/ml2/ml2_conf.ini on ctrl1

```
[ml2]
...snip...
-- l2population is required
mechanism_drivers =openvswitch,l2population
...snip...
```

* Edit /etc/neutron/l3_agent.ini on com1

```
[DEFAULT]
...snip...
agent_mode = dvr_snat
...snip...
```

* Edit /etc/neutron/l3_agent.ini on com2

```
...snip...
agent_mode = dvr
...snip...
```

* Edit /etc/neutron/plugins/ml2/openvswitch_agent.ini on com1 and com2

```
[agent]
...snip...
l2_population = True
enable_distributed_routing = True
...snip...
```

* Restart for changes to take effect

```
-- on ctrl1
systemctl restart neutron-server

-- on com1 and com2
systemctl restart neutron-l3-agent neutron-openvswitch-agent
```

* Verifying neutron services running

```
-- with keystone admin credentials
$ openstack network agent list -c Host -c 'Agent Type' -c Alive -c State
+--------------------+-----------------+-------+-------+
| Agent Type         | Host            | Alive | State |
+--------------------+-----------------+-------+-------+
| Metadata agent     | com2.host.local | True  | UP    |
| Metadata agent     | com1.host.local | True  | UP    |
| L3 agent           | com2.host.local | True  | UP    |
| Open vSwitch agent | com2.host.local | True  | UP    |
| DHCP agent         | com2.host.local | True  | UP    |
| DHCP agent         | com1.host.local | True  | UP    |
| L3 agent           | com1.host.local | True  | UP    |
| Open vSwitch agent | com1.host.local | True  | UP    |
+--------------------+-----------------+-------+-------+
```

### Verifying DVR access flow

* Difference between DVR and non-DVR at VMs access routes

VMs topology and subnet

VM running node | subnet | using router at DVR | using router at non-DVR
-|-|-
same | different | DVR on each VM running node | Network node (running L3 agent node)
different | different | DVR on each VM running node | Network node (running L3 agent node)


```

