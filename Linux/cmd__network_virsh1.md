## Managing libvirt with virsh CLI

Package name: libvirt, libvirt-client

Reference: [Virtualization Deployment and Administration Guide](https://access.redhat.com/documentation/en/)

Libvirt manages the networks to the virtual machines.

virsh command line can manage the networks with libvirt.

The network descriptions are written by XML files.

The commands for managing the networks with virsh CLI

Command | Description
-|-
virsh attach-interface | attach the network interface
virsh detach-interface | detach the network interface
virsh domifstat | get the statistics for domain
virsh iface-bridge | create the bridge and attach the network interface to it
virsh net-autostart | Autostart the network
virsh net-create | create a new network
virsh net-define | only define a new network
virsh net-destroy | stop the network
virsh net-dumpxml | dump the network information in XML format
virsh net-edit | edit the XML network configuration
virsh net-info (network name) | listing the defined attributes for networks
virsh net-list | listing the network names
virsh net-start | start the stopped networks
virsh net-undefine | undefine an inactive network
virsh net-update | update the network configurations

* Practices

```bash

-- write the network description in XML file
# cat <<EOF >test_network.xml
<network><name>test_network</name></network>
EOF

-- create the network
# virsh net-create ./test_network.xml

-- listing the networks
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
 test_network         active     no            no

-- check the details of network
# virsh net-info test_network
Name:           test_network
UUID:           b900f07e-f97c-4756-aedf-7f1eb8fcc422
Active:         yes
Persistent:     no
Autostart:      no
Bridge:         virbr1

-- dump the network description in XML
# virsh net-dumpxml test_network
<network>
  <name>test_network</name>
  <uuid>b900f07e-f97c-4756-aedf-7f1eb8fcc422</uuid>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:e3:03:5b'/>
</network>

-- delete the network
--- stop the network
# virsh net-destory test_network

--- undefine the network
# virsh net-undefine test_network
```

* Control interfaces attatched VMs

```bash

-- detach the interface from the VM
# virsh detach-interface instance-00000005 bridge

-- attach the interface to the VM
# virsh attach-interface instance-00000005 bridge qbrXXX --mac '00:00:00:00:00:00'

```


