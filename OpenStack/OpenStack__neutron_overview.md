## Neutron overview

Neutron is networking component of OpenStack, providing the REST API as user interface.
Neutron network objects as follows,

* Network: L2 broadcast domain
* Subnet: IP ranges to assign the Network object
* Port: Bridge port and instance interface

#### Neutron server
Verifying credentials, requesting the plug-in the tasks, check the quota. 
Neutron server and agents have communication through same message queue service. 

#### Neutron metadata agent
Querying to 169.254.169.254 IP, metadata server providing initial configurations through cloud-init at boot time. 

#### Package: openstack-neutron (from RDO)

Service | Description
-|-
neutron server | providing API service(port: 9696), core daemon
DHCP agent | providing DHCP service to tenant networks
L3 agent | NAT, routing to externel network services

#### L2 services
Neutron provides the Layer 2 networks with a variety of plug-ins.
Tenant networks can be implemented by VLAN, VxLAN, GRE and these ones can be mixed.
Primary Open vSwitch and Open vSwitch with Linux bridge is using for providing L2 networks.
L2 plug-ins should install to compute-node, l3-agent, lbass-agent and any neutron agent services.
-> These L2 plugins were ported to ML2 plugins with driver mechanism.

Package | Description
-|-
openstack-neutron-ml2 | ML2 plugin
openstack-neutron-openvswitch | Open vSwitch plugin
openstack-neutron-linuxbridge | Linux Bridge plugin

L2 agents can be added or removed dynamically, and agents communicate with neutron server in stateless manner.
So agents can be restarted independently.
