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
OS | RHEL 6.5 64bit

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
[   0.0] Opening the source -i ova ./db-c21.ova
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
...snip...
```


