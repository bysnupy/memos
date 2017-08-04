## How to set up a teaming interface with nmcli CLI

### Introduction
For steps to use a teaming interface withou NetworkManager service.

### Environment

Item | Value
-|-
OS | CentOS 7.3
NetworkManager | stopped/disabled
NICs | 2 NICs (eth0, eth1)

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

* Filename: /etc/sysconfig/network-scripts/ifcfg-team0

```ini
# 
DEVICE=team0
DEVICETYPE=Team
TEAM_CONFIG='{"runner": {"name": "lacp"}, "link_watch": {"name": "ethtool"}}'
BOOTPROTO=none
IPADDR=192.168.7.70
PREFIX=24
GATEWAY=192.168.7.1
ONBOOT=yes
NM_CONTROLLED=no
```


