The virtual device name:

## tapXXX
Ethernet frames can be read and written by user spaces process or program to tap devices.
Tap device is L2 device and Ethernet frames are sent by tap devices

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


## tunXXX
IP packet can be received and sent without ethernet frames, and L3 device.

## vethXXX

## macvtapXXX

## virbrXXX

## greXXX
