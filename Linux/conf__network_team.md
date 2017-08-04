## How to set up a teaming interface with nmcli CLI

### Introduction
For steps to use a teaming interface without NetworkManager service.

### Environment

Item | Value
-|-
OS | CentOS 7.3
NetworkManager | stopped/disabled
NICs | 2 NICs (eth0, eth1)

Reference: teamd document samples; /usr/share/doc/teamd-VERSION/example_ifcfgs/

### Configuration steps

Configuration Summary

Item | Value
-|-
Team interface name | team0
Team member NICs | eth0,  eth1
Runner | lacp
Link watcher | ethtool

To create the team0 basic teaming interface, see the steps as follows

#### Step1: Create the team0 ifcfg interface file

* Team master file

```ini
# file path: /etc/sysconfig/network-scripts/ifcfg-team0
DEVICE=team0
DEVICETYPE=Team
TEAM_CONFIG='{"runner": {"name": "lacp"}, "link_watch": {"name": "ethtool"}}'
#TEAM_CONFIG='{"runner": {"name": "lacp", "active": true, "fast_rate": true, "tx_hash": ["eth", "ipv4", "ipv6"]},"link_watch":   {"name": "ethtool"}}'
BOOTPROTO=none
IPADDR=192.168.7.70
PREFIX=24
GATEWAY=192.168.7.1
ONBOOT=yes
NM_CONTROLLED=no
```

* Team Slave files 

```ini
# file path: /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
DEVICETYPE=TeamPort
TEAM_MASTER=team0
ONBOOT=yes
NM_CONTROLLED=no

# file path: /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
DEVICETYPE=TeamPort
TEAM_MASTER=team0
ONBOOT=yes
NM_CONTROLLED=no
```

#### Step2: Link up the team0 interface and verifying the state

```bash
-- starting up the team0 interface
# ifup team0

-- show the team0 connectiviry
# teamctl team0 state
...snip...
```


