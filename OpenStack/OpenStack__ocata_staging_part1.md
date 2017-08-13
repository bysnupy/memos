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
ctrl1.host.local| 4 Cores, Xeon | 8 GiB | 1 NICs | 200 GB, SATA, no RAID
com1.host.local | 12 Cores (HT: 24), Xeon | 64 GiB |3 NICs | 1 TB, SAS, 10 RAID 

* Node interfaces and components information (virtual servers)

Node|Components|Interfaces|Interface roles
----|----------|----------|---------------
ctrl1.host.local|Horizon<br/>Keystone<br/>Glance<br/>Management APIs<br/>MariaDB<br/>RabbitMQ<br/>Memcached | team0: 172.16.9.170 | team0: Management
com1.host.local|Nova<br/>Neutron | eth0: 172.16.9.171<br/>eth1:192.168.33.171<br/>eth2: no IP | eth0: Management<br/>eth1: VM ineternal network (tenant network)<br/>eth2: External network (provider network)
com2.host.local|Nova<br/>Neutron | eth0: 172.16.9.172<br/>eth1:192.168.33.172<br/>eth2: no IP | eth0: Management<br/>eth1: VM ineternal network (tenant network)<br/>eth2: External network (provider network)

* Packstack answer file at the first installation

:warning: check NTP port opening on the iptables rules

```ini
CONFIG_CINDER_INSTALL=n
CONFIG_SWIFT_INSTALL=n
CONFIG_CEILOMETER_INSTALL=n
CONFIG_AODH_INSTALL=n
CONFIG_GNOCCHI_INSTALL=n
CONFIG_COMPUTE_HOSTS=172.16.9.171
CONFIG_NETWORK_HOSTS=172.16.9.171
CONFIG_MARIADB_PW=stage
CONFIG_KEYSTONE_ADMIN_PW=stage
CONFIG_NEUTRON_METERING_AGENT_INSTALL=n
CONFIG_NEUTRON_ML2_TYPE_DRIVERS=flat,vxlan
CONFIG_NEUTRON_ML2_MECHANISM_DRIVERS=openvswitch,l2population
CONFIG_NEUTRON_ML2_VNI_RANGES=10:2000
CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:eth2
CONFIG_NEUTRON_OVS_TUNNEL_IF=eth1
CONFIG_NEUTRON_OVS_TUNNEL_SUBNETS=192.168.33.0/24
CONFIG_PROVISION_DEMO=n
```

* Packstack answer file at the second addtional compute node installation

:warning: You shoud add the iptables rule (/etc/sysconfig/iptables) manually about added the com2 node on the ctrl1 node.

```ini
CONFIG_CINDER_INSTALL=n
CONFIG_SWIFT_INSTALL=n
CONFIG_CEILOMETER_INSTALL=n
CONFIG_AODH_INSTALL=n
CONFIG_GNOCCHI_INSTALL=n
EXCLUDE_SERVERS=172.16.9.170,172.16.9.171
CONFIG_COMPUTE_HOSTS=172.16.9.171,172.16.9.172
CONFIG_NETWORK_HOSTS=172.16.9.171,172.16.9.172
CONFIG_MARIADB_PW=stage
CONFIG_KEYSTONE_ADMIN_PW=stage
CONFIG_NEUTRON_METERING_AGENT_INSTALL=n
CONFIG_NEUTRON_ML2_TYPE_DRIVERS=flat,vxlan
CONFIG_NEUTRON_ML2_MECHANISM_DRIVERS=openvswitch,l2population
CONFIG_NEUTRON_ML2_VNI_RANGES=10:2000
CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:eth2
CONFIG_NEUTRON_OVS_TUNNEL_IF=eth1
CONFIG_NEUTRON_OVS_TUNNEL_SUBNETS=192.168.33.0/24
CONFIG_PROVISION_DEMO=n
```

