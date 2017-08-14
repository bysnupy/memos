## Common network management in the Linux

### Introduction

Taking notes of how to configure the common network interfaces in the Linux.

#### Interface configuration file

The configuration files are located in this path and filename patterns; /etc/sysconfig/network-scripts/ifcfg-DEVICENAME
And active settings located below /sys/class/net/

Interface configuration options

Option | Description
-|-
DEVICE=devicename | network device name
BOOTPROTO=dhcp/static/none | protocols
IPADDRx=yyy.yyy.yyy.yyy | IPv4 adress. you can set up multiple IP addresses, such as IPADDR1, IPADDR2 and so on
PREFIXx=num | Netmask. you should set up it each IP addresses, such as PREFIX1, PREFIX2 and so on
GATEWAY=yyy.yyy.yyy.yyy | Reachable network gateway
DNSx=zzz.zzz.zzz.zzz | name server address, can be mutiple addresses, such as DNS1, DNS2 and so on
ONBOOT=yes/no | yes is to activate the interface at the boot time
HWADDR=00:00:00:00:00:00 | NIC MAC address
MACADDR=00:00:00:00:00:00 | This option is mutually exclusive with HWADDR, it overrides the interface MAC address
NM_CONTROLLED=yes/no | yes means to be controlled by NetworkManager service

#### Interface name nomenclature

Since implemented biosdevname as of RHEL6, depending on the bios device name.

* < RHEL6

Device name | Description
-|-
eth0, eth1 | Ethernet device
wlan0, wlan1 | Wireless device

* >= RHEL6

Device name | Description
-|-
pNpM | PCI N slot and M port
emN | on-board network card

