## Neutron overview

Neutron is networking component of OpenStack, providing the REST API as user interface.
Neutron network objects as follows,

* Network: L2 broadcast domain
* Subnet: IP ranges to assign the Network object
* Port: Bridge port and instance interface

#### Neutron server
Verifying credentials, requesting the plug-in the tasks, check the quota. 
Neutron server and agents have communication through same message queue service.
L2, L3 and DHCP agents are communicate with neutron service through API and message queue like Qpid or AMQP.
These agents are operating independently from neutron service, so can be deployed in multiple nodes.
The agents management can be operate via API and message queue, such as adding, removing and so on.

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

#### VLAN and GRE
Neutron provides VLAN for tenant networks and the interfaces should have 802.1q VLAN trunking support.
Ethernet header is supported by 4096 VLAN IDs.

GRE is the other solution for providing isolated networks, The GRE tunnel is point to point virtual link over IP.
GRE has a slight overhead to IP packet.

#### DHCP agent
The DHCP agent must be installed along with L2 agent because L2 connectivity is needed.
Neutron service is communicated with DHCP agent via message queue.

#### L3 agents
Providing the router and floating IP services.

