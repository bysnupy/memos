## ip command practice

### Introduction
the interface configuration files into /etc/sysconfig/network-scripts/.
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
MACADDR=MAS_ADDRESS

#DHCP ip addressing
BOOTPROTO=dhcp

#Alternative static ip addressing
BOOTPROTO=static
IPADDR=192.168.1.11
PREFIX=24
GATEWAY=192.168.1.1
DNS1=192.168.1.22
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

## ip command

ip command shows the current interface states and IP addresses

```bash
# ip addr show eth0

# ip link show eth0
```

