## Ceph Installation Tutorial

### Introduction
This is the Ceph installation tutorials for memo.

Reference: [official ceph documentation](http://docs.ceph.com/docs/master/)

### Environment

* Version and account informations

Field | Value
-|-
OS | CentOS 7.4.1708
Ceph | Jewel
Deploy OS username | cephnode

* The nodes list

Node | IP | Components 
-|-|-
ceph-admin | 172.16.9.160 | deploy, monitor, metadata server, rados gateway
ceph-osd0 | 172.16.9.161 | osd (object storage device)
ceph-osd1 | 172.16.9.162 | osd (object storage device)
ceph-osd2 | 172.16.9.163 | osd (object storage device)

### Prerequisite settings

* Enabling NTP - All nodes

* Deploy OS user with setting escalation privileges - All nodes

```bash
cat <<EOF >/etc/sudoers.d/cephnode ; chmod 0440 /etc/sudoers.d/cephnode; visudo -c
Defaults:cephnode !requiretty
cephnode ALL=(root) NOPASSWD: ALL
EOF

/etc/sudoers: parsed OK
/etc/sudoers.d/cephnode: parsed OK

```

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
yum install ceph-deploy -y
```

## Step2: Installing the Ceph components

* Create the directory for maintaining ceph configuration files on admin node

```
-- executing as cephnode account
mkdir $HOME/cluster_dir
```

* Create the cluster on admin node

```
ceph-deploy new ceph-admin

[ceph_deploy.conf][DEBUG ] found configuration file at: /root/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /usr/bin/ceph-deploy new ceph-admin
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
...snip...
[ceph_deploy.new][DEBUG ] Monitor initial members are ['ceph-admin']
[ceph_deploy.new][DEBUG ] Monitor addrs are ['172.16.9.160']
[ceph_deploy.new][DEBUG ] Creating a random mon key...
[ceph_deploy.new][DEBUG ] Writing monitor keyring to ceph.mon.keyring...
[ceph_deploy.new][DEBUG ] Writing initial config to ceph.conf...
```

* Modify the ceph.conf file into the cluster_dir directory on admin node

```ini
[global]
..snip...
mon_initial_members = ceph-admin
mon_host = 172.16.9.160
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
# the number of replicas for objects in the pool, default value is 3
osd pool default size = 3
public network = 172.16.9.0/24
```

* Installing the Ceph to all nodes from admin node with ceph-deploy

:warning:Delete the ceph.repo which created for installing ceph-deploy on admin node. <br/>
Because this file was conflicted with ceph-deploy tasks.
```
cd $HOME/cluster_dir
ceph-deploy install --release jewel ceph-admin ceph-osd0 ceph-osd1 ceph-osd2
```




