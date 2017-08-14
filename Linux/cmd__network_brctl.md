## brctl command usage

Package name: bridge-utils

brctl command manage the Linux Bridges that emulte the switch connecting.
When first virutal birdge is created, load the bridge kernel module.
And the traffic is managed into the kernel memory area.

Command|Description
-|-
brctl show| Listing the bridges
brctl showmacs (bridge name) | Listing the MAC addresses for the bridge
brctl addbr (bridge name) | Adding the bridge
brctl delbr (bridge name) | Deleting the bridge
brctl addif (bridge name) (device name) | Adding the interface device to the bridge
brctl delif (bridge name) (device name) | Deleting the interface device from the bridge

## Practices

* Managing with brctl CLI.

```bash
-- create the virtual bridge br01
# brctl addbr br01

# brctl show br01
bridge name     bridge id               STP enabled     interfaces
br01            8000.000000000000       no

-- attaching the interface to bridge
# ip address flush eth0
# brctl addif br01 eth0

-- The bridges that attach internal interfaces can have IP addresses.
# ip address add 192.168.124.108/24 dev br01

# ip link set dev br01 up

# ip route add default via 192.168.124.254

-- ifup and ip link set down
--- up and down
# ifup BRIDGEDEVICENAME
# ip link set down dev BRIDGEDEVICENAME
```

* Configuring persistently

interface config file: /etc/sysconfig/network-scripts/ifcfg-eth0

```ini
DEVICE=eth0
BRIDGE=br01
ONBOOT=yes
```

bridge config file: /etc/sysconfig/network-scripts/ifcfg-br01

```ini
DEVICE=br01
TYPE=Bridge
IPADDR=192.168.124.108
PREFIX=24
GATEWAY=192.168.124.254
ONBOOT=yes
```
