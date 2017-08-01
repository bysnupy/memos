## Neutron overview

Neutron is networking component of OpenStack, providing the REST API as user interface.
Neutron network objects as follows,

* Network: L2 broadcast domain
* Subnet: IP ranges to assign the Network object
* Port: Bridge port and instance interface

#### Neutron server
Verifying credentials, requesting the plug-in the tasks, check the quota. 
Neutron server and agents have communication through same message queue service. 

#### Neutron metadata agent: query to 169.254.169.254 IP, metadata server providing initial configurations through cloud-init at boot time. 

#### Package: openstack-neutron (from RDO)

Service | Description
-|-
neutron server | providing API service(port: 9696), core daemon
DHCP agent | providing DHCP service to tenant networks
L3 agent | NAT, routing to externel network services

#### L2 services

Package | Description
-|-
openstack-neutron-ml2 | ML2 plugin
openstack-neutron-openvswitch | Open vSwitch plugin
openstack-neutron-linuxbridge | Linux Bridge plugin
