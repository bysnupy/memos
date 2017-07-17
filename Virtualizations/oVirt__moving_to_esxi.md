## Moving a VM from oVirt to ESXi

### Introduction
This practice is about the migration of the virtual machine from oVirt or KVM based to ESXi with qemu-img command.

### Environment

Item|Value
-|-
  ESXi| ESXi 6.0 Update 2 Build 3620759
  oVirt(engine and node)| 4.0.4
  qemu-img| qemu-img-1.5.3-105.el7.x86_64
  
### Configuration Summary
This configuration steps assume that 5 servers are existing as follows (It's helpful to reading the related post):

Hostname|Roles
-|-
esxi.host.local| ESXi hypervisor host. (ESXi 6.0 U2)
ovirt.host.local| oVirt hypervisor host. (oVirt node 4.0.4)
nfs.host.local| NFS server for the oVirt Export/Import Storage Domain (':/nfs/shares' is exported shares path) (CentOS 7.2)
conv.host.local| one server for converting the VM image from ESXi format to oVirt format with qemu-img. (CentOS 7.2)
guest.vmhost.local| The guest virtual machine for moving to ESXi. (CentOS 7.2)

In practice any one server can have mutiple roles such as nfs and converting vm image roles can be together on the one server.
I think that the servers just separated by roles are more helpful for understanding and appyling your enviroments.

The process flow across all the servers as follows (Figure 1):

![figure1](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__ovirt2esxi_figure1.png)

### Configuration steps
We assume that moving guest VM(guest.vmhost.local) is existing on the oVirt host server before working for the migration tasks. 
The moving target ESXi host server need to enable the SSH server service and open the SSH client port in advance.

#### Step1: Export the guest VM from oVirt to Export Storage Domain NFS server.
First the moving VM guest server should shutdown before export the VM.
You remember the VM DISK image UUID from 'Disks' tag to search the correct image file for converting. (The UUID is 57b5bf87-4bee-42ab-8910-f7e93419e0dc through this post.)
Select the stopped VM in the 'Virtual Machines' tag and click the 'Export' link.
And then the dialog window appeared for Export process you can complete the tasks according to the instructions on the dialog.

The following image for supplementary explanation.

![step1](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__ovirt2esxi_step1.png)

Step2: Convert the guest VM's image file to VMDK(Virtual Machine Disk) format.
The 'conv.host.local' server should mount the 'nfs.host.local' NFS server without permission problems.

There is a directory tree for the Export Storage NFS Domain.

```bash
|-- [ Export Storage Domain UUID ]
     |-- images
     |   |-- [ Exported virtual machine DISK image UUID] 
     |       |--- [ Exported image UUID ]
     |       |--- [ Exported image UUID ].meta
     |-- master
         |---vms
             |--- [Arbitrary UUID]
                   |--- [Arbitrary UUID].ovf
```

It's the real directory tree at this task step.
You can verify the image file identity from OVF xml file and you can convert the correct raw image file without making mistakes.

```bash
[conv ]# mount -t nfs nfs.host.local:/nfs/shares /mnt
[conv ]# ls -l /mnt
/mnt
|-- b068bd53-60f2-4270-8257-0bc962de834f
     |-- images
     |   |-- 57b5bf87-4bee-42ab-8910-f7e93419e0dc
     |       |--- deb5a3fd-a818-4d6a-82cf-e566756c37b8
     |       |--- deb5a3fd-a818-4d6a-82cf-e566756c37b8.meta
     |-- master
         |---vms
             |--- 480d5d47-e6e6-487f-95c3-286c3ad21fec
                   |--- 480d5d47-e6e6-487f-95c3-286c3ad21fec.ovf
...snip...
[conv ]# cat /mnt/b068bd53-60f2-4270-8257-0bc962de834f/master/vms/480d5d47-e6e6-487f-95c3-286c3ad21fec/480d5d47-e6e6-487f-95c3-286c3ad21fec.ovf |
  xmllint --format - | grep File
<File ovf:href="57b5bf87-4bee-42ab-8910-f7e93419e0dc/deb5a3fd-a818-4d6a-82cf-e566756c37b8" ovf:id="deb5a3fd-a818-4d6a-82cf-e566756c37b8" ovf:size="17179869184" ovf:description="Active VM" ovf:disk_storage_type="IMAGE" ovf:cinder_volume_type=""/>
[conv ]# cd /mnt//b068bd53-60f2-4270-8257-0bc962de834f/images/57b5bf87-4bee-42ab-8910-f7e93419e0dc
[conv ]# cat ./deb5a3fd-a818-4d6a-82cf-e566756c37b8.meta
DOMAIN=b068bd53-60f2-4270-8257-0bc962de834f
CTIME=1482389127
FORMAT=RAW
DISKTYPE=1
LEGALITY=LEGAL
SIZE=33554432
VOLTYPE=LEAF
DESCRIPTION=
IMAGE=57b5bf87-4bee-42ab-8910-f7e93419e0dc
PUUID=00000000-0000-0000-0000-000000000000
MTIME=0
POOL_UUID=
TYPE=SPARSE
EOF
[conv ]# qemu-img info ./deb5a3fd-a818-4d6a-82cf-e566756c37b8
image: ./deb5a3fd-a818-4d6a-82cf-e566756c37b8
file format: raw
virtual size: 16G (17179869184 bytes)
disk size: 1.0G
[conv ]#
```

It's the coverting command with qemu-img as follows: (The options details are referenced by -h or simply executing without no switches.)

```bash
[conv ]# qemu-img convert -p ./deb5a3fd-a818-4d6a-82cf-e566756c37b8 -O vmdk /mnt/guest.vmhost.local.vmdk
(100.00/100%)
[conv ]#
```

#### Step3: Converting the vmdk file to ESXi main data store as the compatiable format.
I remind you one more the prerequisites for this tasks.
You need to add the Export NFS Storage Domain as additional storage on the ESXi host server through vSphere client console. (look below the image.)

![step3](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__ovirt2esxi_step3.png)

And you should do not only starting the SSH server service but also open the SSH client port with FW configuration.

The following commands will be executing on the ESXi host server through SSH terminal.

First, create the directory for deploy the converted the vmdk file having compatible to ESXi.

```bash
login as: root
Using keyboard-interactive authentication.
Password:
The time and date of this login have been sent to the system logs.

VMware offers supported, powerful system administration tools.  Please
see www.vmware.com/go/sysadmintools for details.

The ESXi Shell can be disabled by an administrative user. See the
vSphere Security documentation for more information.
[root@esxi:~] mkdir /vmfs/volumes/datastore1/guest.vmhost.local.disk
[root@esxi:~]
```

In succession we convert the vmdk format file from the oVirt to ESXi compatible vmdk image file as the VMFS thin-provisioned format.

```bash
[root@esxi:~] vmkfstools -i /vmfs/volumes/nfs_store/guest.vmhost.local.vmdk \
                                   /vmfs/volumes/datastore1/guest.vmhost.local.disk/guest.vmhost.local.vmdk -d thin
Destination disk format: VMFS thin-provisioned
Cloning disk '/vmfs/volumes/nfs_store/guest.vmhost.local.vmdk'...
Clone: 100% done.
[root@esxi:~]
```

#### Step4: Register the virtual machine with converted vmdk image file.
The last step for the VM migration from oVirt to ESXi is registering the ESXi guest virtual machine with same name attached converted vmdk image file. 

Create new virtual machine attached the converted vmdk image with same name as follows:

![step4](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__ovirt2esxi_step4.png)

Let's check the moved virtual machine from oVirt to ESXi.

![step4_1](https://github.com/bysnupy/memos/blob/master/Virtualizations/images/ovirt__ovirt2esxi_step4_1.png)

Done.
