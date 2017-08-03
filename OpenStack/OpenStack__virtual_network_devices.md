## The virtual device names

### Logical diagram

![architecture](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__virtual_network_devices1.png)

### Network namespaces
This name spaces can contain the virtual network interfaces, 
applications started within a namespace will only see the interfaces in that space.
The namespaces have each routing table and own /proc/net.

* Practices

```bash
# ip netns add nsnet1
# ip netns list
nsnet1

# ip netns exec nsnet1 ip addr show
...snip...

# ip netns exec nsnet1 ls -l /proc/net

# ip link add veth-a type veth peer name veth-b
# ip link set veth-b netns nsnet1
# ip netns exec nsnet1 ip addr add 192.168.200.1/24 dev veth-b
# ip netns exec nsnet1 ip link set up dev veth-b
# ip addr add 192.168.200.2/24 dev veth-a
# ip link set up dev veth-a
# ping -c3 192.168.200.1
...snip...
# ip netns exec nsnet1 ping -c3 192.168.200.2
...snip...
# tunctl -t tap1 or ip tuntap tap1 mode tap
# ip link set tap1 netns nsnet1
# ip netns exec nsnet1 ip addr add 192.168.100.1/24 dev tap1
# ip netns exec nsnet1 ip link set up dev tap1
# ip route add 192.168.100.0/24 dev veth-a
# sysctl -w net.ipv4.ip_forward=1
# ping -c3 192.168.100.1
```

### tapXXX
Ethernet frames can be read and written by user spaces process or program to tap devices.
Tap device is L2 device and Ethernet frames are sent by tap devices.

* Tap devices can create with tunctl command into tunctl package. (RHEL6) - ( openvpn –mktun –dev TAPNAME )
* Tap devices can create with ip command, ip tuntap add TAPNAME mode tap. (RHEL7)
* Tap devices can create with ovs-vsctl add-port BRIDGENAME TAPNAME -- set Interface TAPNAME type=internal
* udevadm monitor command can monitor about adding new hardware or virtual devices base on kernel events. 
* Practices

```bash
# tunctl -t tap1
# ip link set dev tap1 up
# ip addr add 192.168.124.7/24 dev tap1
```

### tunXXX
IP packet can be received and sent without ethernet frames, and L3 device.

### vethXXX
veth is network device pair, this pair can connect other virtual network, 
such as two bridge devices could be connected by adding each the veth devices.
veth is virtual patch cable.

Interface name | Description
-|-
qvo | veth pair open vswitch side
qvb | veth pair bridge side

* Practices

```bash
-- specify the both endpoints
# ip link add veth1 type veth peer name veth2
```

### macvtapXXX
MacVTap devices provides virtual network interfaces overlayed a physical network interfaces.
Each virtual network interfaces have own MAC addresses.
MacVTap is based on kernel module macvtap.
MacVTap has four operation modes; Bridge, Private, VEPA, Passthru.

### virbrXXX

### greXXX
