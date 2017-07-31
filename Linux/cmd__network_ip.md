## ip command practice

### Introduction

Package name: iproute

the interface configuration files into /etc/sysconfig/network-scripts/
the interface active settings were shown /sys/class/net/
the interfaca configuration files were named ifcfg-IFNAME

Network interface name has rules:

Ethernet card: eth0 ... ethX
Wireless card: wlan0 ... wlanX

but from 6.1

the biosdevname command refer the interface name from BIOS

PCI X slot Y port: pXpY

Device depended on: emX

### Configuration files

```bash
# vim /etc/sysconfig/network-scripts/ifcfg-eth0

#Device Control
DEVICE=eth0
ONBOOT=yes
NM_CONTROLLED=yes
MACADDR=MAS_ADDRESS - changeable
HWADDR=DEVICE_ADDRESS

#DHCP ip addressing
BOOTPROTO=dhcp

#Alternative static ip addressing
BOOTPROTO=static
IPADDR=192.168.1.11
PREFIX=24
GATEWAY=192.168.1.1
DNS1=192.168.1.22

# Default MTU
MTU=1500
```

### NetworkManager

Default service on RHEL6, and if you need to manage the configuration files individually you shoud NM_CONTROLLED=no

You can start and stop manually the interfaces with ifup and ifdown command.

```bash
-- stop
# ifdown eth0

-- start
# ifup eth0
-- restart
# ifdown eth0; ifup eth0

-- or if you need to restart all interfaces as follows
# systemctl restart network or service network restart

```

### ip command

ip command shows the current interface states and IP addresses

* ip link

ip link : modify L2 settings, such as MAC addresses, promiscuous mode and interface up and down
ip link show 
ip link set DEVICE up | down
ip link set DEVICE promisc on | off
ip link set DEVICE address ADDRESS

* ip address

ip address : modify L3 settins, such as IP addresses
ip address show dev DEVICE
ip address show flush dev DEVICE
ip address add IPADDR dev DEVICE
ip address del IPADDR dev DEVICE

* ip route

ip route add default via IPADDR

* ip about virtual devices

ip link add ( link DEVICE ) ( name ) NAME type TYPE ( ARGS )
TYPE := ( bridge | can | dummy | ifb | ipoib | macvlan | vcan | veth | vlan | vxlan | ip6tnl | ipip | sit )

* ip netns

ip ( OPTIONS ) netns  ( COMMAND | help )

ip netns ( list )
ip netns ( add | delete ) NETNSNAME
ip netns identify PID
ip netns pids NETNSNAME
ip netns exec NETNSNAME command ...
ip netns monitor
 
ip link set netns PID|NETSNAME

* Practices

```bash
# ip -4 addr show eth0

# ip link show eth0

# ip address flush eth0

# ip address add 192.168.123/24 dev eth0

# ip link set dev eth0 up

# ip route add default via 192.168.123.254
```
### ethtool command

check the device type

```bash
# ethtool -i eth0
```
