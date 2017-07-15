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

![components process flow 1](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__esxi2ovirt_flow1.png)

### Configuration steps
In advance the virtual machine should instatall minimally as a guest OS to ESXi because it can minimize the converting process time at this demonstration. At real work you would take account of the target virtual machine size because of the processing time.

#### Step1: Export the virtual machine from ESXi with vSphere management console.
The target guest VM should be stopped to export to OVA file.

The operation example is as follows on the vSphere client.

![step 1](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__esxi2ovirt_step1.png)

The exported OVA file is named 'guest.vmhost.local.ova' as default if the export task has completed.
And then we should transfer the OVA file to the 'conv.host.local' server to convert the OVA file to oVirt format. 
