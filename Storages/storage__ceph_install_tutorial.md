## Ceph Installation Tutorial

### Introduction
This is the Ceph installation tutorials for memo.

Reference: [official ceph documentation](http://docs.ceph.com/docs/master/)

### Environment

* Version and account informations

Field | Value
-|-
OS | CentOS 7.4.1708
Ceph | Jewel, 10.2.9
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

[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephnode/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /bin/ceph-deploy install --release jewel ceph-admin ceph-osd0 ceph-osd1 ceph-osd2
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  testing                       : None
...snip...
[ceph-osd2][DEBUG ] Complete!
[ceph-osd2][INFO  ] Running command: sudo ceph --version
[ceph-osd2][DEBUG ] ceph version 10.2.9 (2ee413f77150c0f375ff6f10edd6c8f9c7d060d0)
```

* Initiate the MON(monitor)

```
cd $HOME/cluster_dir
ceph-deploy mon create-initial
```

* Check the disks for using Object Storage Devices

```
ceph-deploy disk list ceph-osd{0..2}

[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephnode/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /bin/ceph-deploy disk list ceph-osd0 ceph-osd1 ceph-osd2
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  subcommand                    : list
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x1297ea8>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  func                          : <function disk at 0x128cf50>
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  disk                          : [('ceph-osd0', None, None), ('ceph-osd1', None, None), ('ceph-osd2', None, None)]
...snip...
[ceph-osd2][DEBUG ] /dev/vda :
[ceph-osd2][DEBUG ]  /dev/vda2 other, LVM2_member
[ceph-osd2][DEBUG ]  /dev/vda1 other, xfs, mounted on /boot
[ceph-osd2][DEBUG ] /dev/vdb other, unknown
```

* Create the OSD devices on the OSD nodes (with vdb disk)

```
cd $HOME/cluster_dir
ceph-deploy osd create ceph-osd0:vdb ceph-osd1:vdb ceph-osd2:vdb

[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephnode/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /bin/ceph-deploy osd create ceph-osd0:vdb ceph-osd1:vdb ceph-osd2:vdb
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  block_db                      : None
[ceph_deploy.cli][INFO  ]  disk                          : [('ceph-osd0', '/dev/vdb', None), ('ceph-osd1', '/dev/vdb', None), ('ceph-osd2', '/dev/vdb', None)]
...snip...
[ceph-osd2][INFO  ] checking OSD status...
[ceph-osd2][DEBUG ] find the location of an executable
[ceph-osd2][INFO  ] Running command: sudo /bin/ceph --cluster=ceph osd stat --format=json
[ceph_deploy.osd][DEBUG ] Host ceph-osd2 is now ready for osd use.
```

* Copy the admin keyring and configuration file to all nodes

```
cd $HOME/cluster_dir
ceph-deploy admin ceph-admin ceph-osd0 ceph-osd1 ceph-osd2
sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

#### Step3: Check the installation

* Check the cluster status

```
ceph -s
   cluster 8a30216f-cb95-4f13-bd81-5f638cc0e71c
    health HEALTH_OK
    monmap e1: 1 mons at {ceph-admin=172.16.9.160:6789/0}
           election epoch 3, quorum 0 ceph-admin
    osdmap e14: 3 osds: 3 up, 3 in
           flags sortbitwise,require_jewel_osds
     pgmap v28: 64 pgs, 1 pools, 0 bytes data, 0 objects
           100 MB used, 15226 MB / 15326 MB avail
                 64 active+clean
```

* Check the cluster hierarchy

```
ceph osd tree

ID WEIGHT  TYPE NAME          UP/DOWN REWEIGHT PRIMARY-AFFINITY
-1 0.01469 root default
-2 0.00490     host ceph-osd0
 0 0.00490         osd.0           up  1.00000          1.00000
-3 0.00490     host ceph-osd1
 1 0.00490         osd.1           up  1.00000          1.00000
-4 0.00490     host ceph-osd2
 2 0.00490         osd.2           up  1.00000          1.00000
```

* Get the OSDs informations

```
ceph osd dump
epoch 14
fsid 8a30216f-cb95-4f13-bd81-5f638cc0e71c
created 2017-09-15 13:13:23.738720
modified 2017-09-15 13:30:32.573786
flags sortbitwise,require_jewel_osds
pool 0 'rbd' replicated size 3 min_size 2 crush_ruleset 0 object_hash rjenkins pg_num 64 pgp_num 64 last_change 1 flags hashpspool stripe_width 0
max_osd 3
osd.0 up   in  weight 1 up_from 4 up_thru 13 down_at 0 last_clean_interval [0,0) 172.16.9.161:6800/2140 172.16.9.161:6801/2140 172.16.9.161:6802/2140 172.16.9.161:6803/2140 exists,up 294f4279-0afb-4e23-8c8c-f93cc5e3db37
osd.1 up   in  weight 1 up_from 8 up_thru 13 down_at 0 last_clean_interval [0,0) 172.16.9.162:6800/2031 172.16.9.162:6801/2031 172.16.9.162:6802/2031 172.16.9.162:6803/2031 exists,up 9babdb6a-54d2-47c0-9a50-f69225ed986c
osd.2 up   in  weight 1 up_from 12 up_thru 13 down_at 0 last_clean_interval [0,0) 172.16.9.163:6800/1919 172.16.9.163:6801/1919 172.16.9.163:6802/1919 172.16.9.163:6803/1919 exists,up 8077ac00-195a-474b-8eab-cef51be1b6b3
```

#### Step4: Add metadata server

* MDS add to admin node

```
cd $HOME/cluster_dir
ceph-deploy mds create ceph-admin
```

* Check the MDS status
```
ceph mds stat
```

* Create the pools for CephFS

```
ceph osd pool create cephfs_data_pool 64
pool 'cephfs_data_pool' created

ceph osd pool create cephfs_meta_pool 64
pool 'cephfs_meta_pool' created
```

* Create the CephFS

```
ceph fs new cephfs cephfs_meta_pool cephfs_data_pool
new fs with metadata pool 2 and data pool 1
```

#### Step4: Add the RADOSGW

```
cd $HOME/cluster_dir
ceph-deploy rgw create ceph-admin

[ceph_deploy.conf][DEBUG ] found configuration file at: /home/cephnode/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /bin/ceph-deploy rgw create ceph-admin
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  rgw                           : [('ceph-admin', 'rgw.ceph-admin')]
...snip...
[ceph-admin][INFO  ] Running command: sudo systemctl enable ceph.target
[ceph_deploy.rgw][INFO  ] The Ceph Object Gateway (RGW) is now running on host ceph-admin and default port 7480
```

