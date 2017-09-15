## Ceph Installation Tutorial

### Introduction
This is the Ceph installation tutorials for memo.

Reference: official ceph documentation [ceph docs](http://docs.ceph.com/docs/master/)

### Environment

Field | Value
OS | CentOS 7.4.1708
Ceph | Jewel
Deploy OS username | cephnode

Node | IP | Components 
-|-
ceph-admin | 172.16.3.160 | deploy, monitor, metadata server, rados gateway
ceph-osd0 | 172.16.3.161 | osd (object storage device)
ceph-osd1 | 172.16.3.162 | osd (object storage device)
ceph-osd2 | 172.16.3.163 | osd (object storage device)

### Prerequisite settings

* Enabling NTP - All nodes

* Deploy OS user with setting escalation privileges - All nodes

'''bash
cat <<EOF >/etc/sudoers.d/cephnode ; chmod 0440 /etc/sudoers.d/cephnode; visudo -c
Defaults:cephnode !requiretty
cephnode ALL=(root) NOPASSWD: ALL
EOF

/etc/sudoers: parsed OK
/etc/sudoers.d/cephnode: parsed OK

'''

* SSH public key configuration (Admin node -> OSD nodes as cephnode user)

### Installation Steps

## Step1: Installation of ceph-deploy on admin node

The following tasks are conducted on the admin node (ceph-admin).

```
-- executing as root account
yum install -y epel-release && yum update epel-release

-- create a ceph repository file
cat <<EOF > /etc/yum.repos.d/ceph.repo
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1
EOF

-- update all repository
yum clean all && yum update -y

-- install ceph-deploy
yum install ceph-deploy
```






