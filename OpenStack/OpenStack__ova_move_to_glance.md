## ESXi guest vm to Glance as QCOW2 format

### Introduction

Practical procedures for moving ESXi guest OS to Glance storage as QCOW2 format.

### Encountered Bugs

:warning: ([Bug 1374232](https://bugzilla.redhat.com/show_bug.cgi?id=1374232))
selinux relabel fails on RHEL 6.2 guests with "libguestfs error: selinux_relabel: : Success"

I ran into this bug at relabeling selinux contexts during converting of VM image format.
This is not virt-v2v tool's bug, just OS bug.

* My environment is as follows:

Field | Value
-|-
Origin | ESXi guest OS
OS | RHEL 6.5 64bit
DISK | sda,sdb,sdc,sdd

* The bug details:

This bug is due to just typing miss.

File path: /etc/selinux/targeted/contexts/files/file_contexts
Bug place:

```
/var/run/spice-vdagentd.\pid    --      system_u:object_r:vdagent_var_run_t:s0
-> /var/run/spice-vdagentd\.pid    --      system_u:object_r:vdagent_var_run_t:s0
```

* Solutions:

Update following packages (>=6.8)

```
selinux-policy-targeted
selinux-policy
```

### Task steps

* Converting OVA to QCOW2 with moving glance image storage.

```
-- source the admin credentials of OpenStack and 
# source ~/adminrc-openstack
# export LIBGUESTFS_BACKEND=direct
-- converting tasks with virt-v2v command
# virt-v2v -i ova ./OVAIMAGENAME.ova -o glance -of qcow2
[   0.0] Opening the source -i ova ./rhel65.ova
[  18.7] Creating an overlay to protect the source from being modified
[  21.5] Initializing the target -o glance
[  24.0] Opening the overlay
[  27.8] Inspecting the overlay
[  37.1] Checking for sufficient free disk space in the guest
[  37.1] Estimating space required on target for each disk
[  37.1] Converting Red Hat Enterprise Linux Server release 6.5 (Santiago) to run on KVM
virt-v2v: This guest has virtio drivers installed.
[  94.0] Mapping filesystem data to avoid copying unused and blank areas
[  96.6] Closing the overlay
[  96.7] Checking if the guest needs BIOS or UEFI to boot
[  96.7] Assigning disks to buses
[  96.7] Copying disk 1/4 to /var/tmp/glance.JYUnmb/sda (qcow2)
    (100.00/100%)
[ 350.1] Copying disk 2/4 to /var/tmp/glance.JYUnmb/sdb (qcow2)
    (100.00/100%)
[ 598.5] Copying disk 3/4 to /var/tmp/glance.JYUnmb/sdc (qcow2)
    (100.00/100%)
[ 917.8] Copying disk 4/4 to /var/tmp/glance.JYUnmb/sdd (qcow2)
    (100.00/100%)
[1037.7] Creating output metadata
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| architecture     | x86_64                               |
| checksum         | f230602175740907e58d3de9aa832593     |
| container_format | bare                                 |
| created_at       | 2017-08-24T10:15:51Z                 |
| disk_format      | qcow2                                |
| hw_disk_bus      | virtio                               |
| hw_vif_model     | virtio                               |
| hypervisor_type  | kvm                                  |
| id               | 7a8114db-f5ac-483d-8b5b-2d0ad5df27f0 |
| min_disk         | 0                                    |
| min_ram          | 4096                                 |
| name             | rhel65                               |
| os_distro        | rhel                                 |
| os_type          | linux                                |
| os_version       | 6.5                                  |
| owner            | 5d49cbafb5504c2d8fa141ed3452878c     |
| protected        | False                                |
| size             | 30936203264                          |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2017-08-24T10:24:03Z                 |
| virtual_size     | None                                 |
| visibility       | shared                               |
| vm_mode          | hvm                                  |
+------------------+--------------------------------------+
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| architecture     | x86_64                               |
| checksum         | 49001911a167e5623ddb79f12efc270d     |
| container_format | bare                                 |
| created_at       | 2017-08-24T10:24:06Z                 |
| disk_format      | qcow2                                |
| hw_disk_bus      | virtio                               |
| hw_vif_model     | virtio                               |
| hypervisor_type  | kvm                                  |
| id               | 0823deaa-8459-4176-a929-522d7d6a262d |
| min_disk         | 0                                    |
| min_ram          | 4096                                 |
| name             | rhel65-disk2                         |
| os_distro        | rhel                                 |
| os_type          | linux                                |
| os_version       | 6.5                                  |
| owner            | 5d49cbafb5504c2d8fa141ed3452878c     |
| protected        | False                                |
| size             | 21062942720                          |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2017-08-24T10:30:18Z                 |
| virtual_size     | None                                 |
| visibility       | shared                               |
| vm_mode          | hvm                                  |
+------------------+--------------------------------------+
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| architecture     | x86_64                               |
| checksum         | 61a0f768cf5bc3179bd9a6dc4a789bea     |
| container_format | bare                                 |
| created_at       | 2017-08-24T10:30:22Z                 |
| disk_format      | qcow2                                |
| hw_disk_bus      | virtio                               |
| hw_vif_model     | virtio                               |
| hypervisor_type  | kvm                                  |
| id               | 2c731d08-343c-442c-9980-e4aa328ec823 |
| min_disk         | 0                                    |
| min_ram          | 4096                                 |
| name             | rhel65-disk3                         |
| os_distro        | rhel                                 |
| os_type          | linux                                |
| os_version       | 6.5                                  |
| owner            | 5d49cbafb5504c2d8fa141ed3452878c     |
| protected        | False                                |
| size             | 25847398400                          |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2017-08-24T10:39:38Z                 |
| virtual_size     | None                                 |
| visibility       | shared                               |
| vm_mode          | hvm                                  |
+------------------+--------------------------------------+
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| architecture     | x86_64                               |
| checksum         | 6122ede6d8021d9556cc6e816a4d08f9     |
| container_format | bare                                 |
| created_at       | 2017-08-24T10:39:41Z                 |
| disk_format      | qcow2                                |
| hw_disk_bus      | virtio                               |
| hw_vif_model     | virtio                               |
| hypervisor_type  | kvm                                  |
| id               | c466df7e-c16f-41a6-a91e-f1ef4e4ec98a |
| min_disk         | 0                                    |
| min_ram          | 4096                                 |
| name             | rhel65-disk4                         |
| os_distro        | rhel                                 |
| os_type          | linux                                |
| os_version       | 6.5                                  |
| owner            | 5d49cbafb5504c2d8fa141ed3452878c     |
| protected        | False                                |
| size             | 10456137728                          |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2017-08-24T10:42:37Z                 |
| virtual_size     | None                                 |
| visibility       | shared                               |
| vm_mode          | hvm                                  |
+------------------+--------------------------------------+
[2648.9] Finishing off

-- check image list
# openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
...snip...
| 7a8114db-f5ac-483d-8b5b-2d0ad5df27f0 | db-c21       | active |
| 0823deaa-8459-4176-a929-522d7d6a262d | db-c21-disk2 | active |
| 2c731d08-343c-442c-9980-e4aa328ec823 | db-c21-disk3 | active |
| c466df7e-c16f-41a6-a91e-f1ef4e4ec98a | db-c21-disk4 | active |
...snip...
+--------------------------------------+--------------+--------+
```

### Conclusion

Multiple disks need to convert volumes for launching instance with it.
So I should add the cinder or swift services to the OpenStack.
