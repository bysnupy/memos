## Moving a VM from ESXi to oVirt

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

#### Step2: Convert and transfer the guest VM to oVirt Export Storage Domain.
You might be encountered the following errors while convert processing with virt-v2v.

Common error1: It's a permission error if you process as root account.

```bash
virt-v2v: error: libguestfs error: could not create appliance through
libvirt.

Try running qemu directly without libvirt using this environment variable:
export LIBGUESTFS_BACKEND=direct

Original error from libvirt: Cannot access backing file
'/var/tmp/ova.vuHiQ6/guest.vmhost.local-disk1.vmdk' of storage file
'/var/tmp/v2vovl67af1b.qcow2' (as uid:107, gid:107): Permission denied
[code=38 domain=18]

If reporting bugs, run virt-v2v with debugging enabled and include the
complete output:

  virt-v2v -v -x [...]
```

This error is caused by using libvirt backend as 'qemu' account ([detailed reference link](http://libguestfs.org/guestfs-faq.1.html#permission-denied-when-running-libguestfs-as-root)). I recommend directly processing the converting as root account without libvirtd bacause it can avoid the errors about permission around libvirt, if your environment is allowed using root account to convert the VM. 
It specify the 'LIBGUESTFS_BACKEND=direct' to process the task directly without libvirtd.

```bash
# LIBGUESTFS_BACKEND=direct virt-v2v -i ova ./guest.vmhost.local.ova ...snip...
```

Common error2: The NFS access permission error.

```bash
virt-v2v: error: mount command failed, see earlier errors.

This probably means you didn't specify the right Export Storage Domain path
[-os nfs.host.local:/nfs/shares], or else you need to rerun virt-v2v as
root.

If reporting bugs, run virt-v2v with debugging enabled and include the
complete output:

  virt-v2v -v -x [...]
```

This error is caused by not adding the 'conv.host.local' server to NFS access allowed list.
You should allow the 'conv.host.local' server to be accessible to 'nfs.host.local' NFS server before converting.

I started executing the 'virt-v2v' command with the following options. ([options reference link](http://libguestfs.org/virt-v2v.1.html))

```bash
[conv ]# LIBGUESTFS_BACKEND=direct virt-v2v -i ova ./guest.vmhost.local.ova \
                                            -o rhev -of qcow2 -os nfs.host.local:/nfs/shares -v -x
virt-v2v: libguestfs 1.28.1 (x86_64)
[   0.0] Opening the source -i ova ./guest.vmhost.local.ova
tar -xf './guest.vmhost.local.ova' -C '/var/tmp/ova.ZflJ8V'
source name: guest.vmhost.local
hypervisor type: vmware
memory: 2147483648 (bytes)
nr vCPUs: 2
CPU features:
firmware: bios
display:
sound:
...snip...
[ 116.0] Creating output metadata
[ 116.0] Finishing off
umount '/tmp/v2v.TZcelY'
OVF:
<ovf:envelope ovf:version="0.9" xmlns:ovf="http://schemas.dmtf.org/ovf/envelope/1/" xmlns:rasd="http://schemas.dmtf.org/wbem/wscim/1/cim-schema/2/CIM_ResourceAllocationSettingData" xmlns:vssd="http://schemas.dmtf.org/wbem/wscim/1/cim-schema/2/CIM_VirtualSystemSettingData" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <references>
    <file ovf:description="Exported by virt-v2v 1.28.1" ovf:href="25534d4f-d07d-459d-a9ef-3ed733cbcdef/90b62105-4ae8-4a4b-826b-dfe970ae28d9" ovf:id="90b62105-4ae8-4a4b-826b-dfe970ae28d9" ovf:size="17179869184">
  </file></references>
  </ovf:envelope>
<section xsi:type="ovf:NetworkSection_Type">
    <info>List of networks</info>
    <network ovf:name="Ethernet 1">
  </network></section>
  <section xsi:type="ovf:DiskSection_Type">
    <info>List of Virtual Disks</info>
    <disk ovf:actual_size="2" ovf:boot="True" ovf:disk-interface="VirtIO" ovf:disk-type="System" ovf:diskid="90b62105-4ae8-4a4b-826b-dfe970ae28d9" ovf:fileref="25534d4f-d07d-459d-a9ef-3ed733cbcdef/90b62105-4ae8-4a4b-826b-dfe970ae28d9" ovf:format="http://en.wikipedia.org/wiki/Byte" ovf:parentref="" ovf:size="16" ovf:vm_snapshot_id="61569757-5a3d-4342-b470-6dd174d8edab" ovf:volume-format="COW" ovf:volume-type="Sparse">
  </disk></section>
  <content ovf:id="out" xsi:type="ovf:VirtualSystem_Type">
    <name>guest.vmhost.local</name>
    <templateid>00000000-0000-0000-0000-000000000000</templateid>
    <templatename>Blank</templatename>
...snip...
        <rasd:instanceid>3de59667-835c-41cf-b3dc-a00684e3a4e2</rasd:instanceid>
        <rasd:caption>Ethernet adapter on Ethernet 1</rasd:caption>
        <rasd:resourcetype>10</rasd:resourcetype>
        <rasd:resourcesubtype>3</rasd:resourcesubtype>
        <type>interface</type>
        <rasd:connection>Ethernet 1</rasd:connection>
        <rasd:name>eth0</rasd:name>
      </content>
[conv ]#
```

#### Step3: Import the VM on the Export Storage Domain to the oVirt.

Lastly we import the VM into oVirt with oVrit web management console and startup the VM as guest virtual machine.
We would log into the oVirt web console and then click the 'Storage' tag, you can see your Export NFS Storage Domain.
It would appear the details below the window after selecting the Export Storage Domain.
Click the 'Import' sub tags, I'll be represented the dialog about Import details menu.
You can Import with optional details (such as DISK allocation method and so on) the VM through the dialog. 

It's the image about above process flows.

![step3](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__esxi2ovirt_step3.png)

Addtionally you might configure the VM about installing agent package, networking setup and so on.

Done.
