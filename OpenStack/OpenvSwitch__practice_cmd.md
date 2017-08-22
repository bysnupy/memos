## Open vSwitch control with CLI

Package name: openvswitch (from RDO repository)

Open vSwitch is providing additional functionality, such as VLAN (IEEE 802.1Q), VxLAN, GRE, NetFlow, OpenFlow, SPAN, RSPAN, LACP, STP  protocols.

Open vSwitch can operate in flow mode based on rules.

Extra knowledge, OpenFlow implements remotely access to the flow tables on the routers.

### ovs-vsctl usage

Command|Description
-|-
ovs-vsctl show | show the database overview
ovs-vsctl list-br | listing the bridges
ovs-vsctl list-ports (bridge name) | listing all port names on the specific bridge devices
ovs-vsctl list-ifaces (bridge name) | listing the interfaces on the specific bridge devices
ovs-vsctl iface-to-br (interface name) | listing the bridges which included specific interfaces
ovs-vsctl add-br (bridge name) | create the new virtual bridge
ovs-vsctl del-br (bridge name) | delete bridge and related all ports
ovs-vsctl add-port (bridge name) (port name) | add the port device to the bridge
ovs-vsctl del-port (bridge name) (port name) | delete the port from bridge

### Practices

```bash
-- enabling and starting the Open vSwitch service
# systemctl enable openvswitch
# systemctl start openvswitch

-- create the new bridge
# ovs-vsctl add-br ovs-br1

-- the interface L3 settings should flush to attach the bridge
# ip address flush dev enp0s8

-- assign the IP address to the bridge
# ip address add 192.168.124.109/24 dev ovs-br1

-- attach the interface to the bridge
# ovs-vsctl add-port ovs-br1 enp0s8

-- bring the bridge up
# ip link set dev ovs-br1 up

-- check the settings
a4deced9-ec05-414e-a15d-acf1bb87fa45
    Bridge "ovs-br1"
        Port "enp0s8"
            Interface "enp0s8"
        Port "ovs-br1"
            Interface "ovs-br1"
                type: internal
    ovs_version: "2.6.1"

-- check the L3 configurations
# ip address show ovs-br1

-- setting up the default gateway
# ip route add default via 192.168.124.254

-- check the flow mode
# ovs-ofctl dump-flows ovs-br1
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=606.934s, table=0, n_packets=8, n_bytes=648, idle_age=492, priority=0 actions=NORMAL

-- show a specific bridge details
# ovs-vsctl show bridge BRIDGENAME

-- set attributes
# ovs-vsctl set bridge BRIDGENAME ATTRIBUTENAME:KEY=VALUE
```

### ovs-ofctl usage

ovs-ofctl command is used by managing and monitoring Open vSwitch and OpenFlow switches.

```
-- display database overview
# ovs-ofctl show  

-- check network ports
# ovs-ofctl dump-ports-desc

-- display a flow table
# ovs-ofctl dump-flows
```
### Interface configuration for persistent

```ini
# /etc/sysconfig/network-scripts/ifcfg-br-ovs0
DEVICE=br-ovs0
DEVICETYPE=ovs
TYPE=OVSBridge
BOOTPROTO=static
IPADDR=yyy.yyy.yyy.yyy
NETMASK=255.255.255.0
MACADDR=00:00:00:00:00:00
OVS_EXTRA="set bridge br-ovs0 other-config:hwaddr=$MACADDR"
ONBOOT=yes

# /etc/sysconfig/network-scripts/ifcfg-eth2
DEVICE=eth2
TYPE=OVSPort
DEVICETYPE=ovs
OVS_BRIDGE=br-ovs0
ONBOOT=yes


```

### ovs-dpctl usage

```bash
-- display the data paths on the switch
# ovs-dpctl show
```
