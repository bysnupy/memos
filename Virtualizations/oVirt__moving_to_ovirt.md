## Moving VM from ESXi to oVirt

### Introduction
It's a example for moving VM from ESXi to oVirt(upstream project of RHEV) with virt-v2v.

### Environment

Item|Value
-|-
  ESXi| ESXi 6.0 Update 2 Build 3620759
  oVirt(engine and node)| 4.0.4
  virt-v2v| virt-v2v-1.28.1-1.55.el7.x86_64

### Configuration Summary
This configuration steps assume that 5 servers are existing as follows:

* esxi.host.local: ESXi hypervisor host. (ESXi 6.0 U2)
* ovirt.host.local: oVirt hypervisor host. (oVirt node 4.0.4)
* nfs.host.local: NFS server for the oVirt Export/Import Storage Domain (':/nfs/shares' is exported shares path) (CentOS 7.2)
* conv.host.local: one server for converting the VM image from ESXi format to oVirt format with virt-v2v. (CentOS 7.2)
* guest.vmhost.local: The guest virtual machine for moving to oVirt. (CentOS 7.2)

In practice any one server can have mutiple roles such as nfs and converting vm image roles can be together on the one server.
I think that the servers just separated by roles are more helpful for understanding and appyling your enviroments.

The process flow across all the servers as follows(Figure 1):
