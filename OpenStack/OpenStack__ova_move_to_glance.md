## ESXi guest vm to Glance as QCOW2 format

### Introduction

Practical procedures for moving ESXi guest OS to Glance storage as QCOW2 format.

#### Steps

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




