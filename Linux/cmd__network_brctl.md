## brctl command usage

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
