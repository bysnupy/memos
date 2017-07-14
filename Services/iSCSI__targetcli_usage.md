## iSCSI target configuration with targetcli command

### Introduction
As of RHEL7/CentOS7, the iSCSI target configuration command has been changed, I wrote out my practices here.
I thought that the new one is so interesting.

### Environment
Item|Value
-|-
OS| CentOS 7.2.1511
Packages| targetcli-2.1.fb41-3.el7.noarch (for target configuration)<br/>iscsi-initiator-utils-6.2.0.873-33.el7_2.2.x86_64 (for initiator configuration)

### Configuration steps
Firstly configure the iSCSI target server, and then configure the client initiator server.
The target configurations are required two virtual disks(1GB) and 1 image file(1GB) here.

#### Step1: The prerequisite tasks for the target configuration

The iSCSI stand for Internet Small Computer System Interface, the target serve the storage device over the network.
The target meaning like a server on the server-client model.

It'll be enabled and started

```bash
# systemctl is-enabled target.service
disabled
# systemctl enable target.service
Created symlink from /etc/systemd/system/multi-user.target.wants/target.service to /usr/lib/systemd/system/target.service.
# systemctl start target
# firewall-cmd --permanent --add-service=iscsi-target
success
# firewall-cmd --reload
success
```

#### Step2: the target main configurations with targetcli command

We are going to configure the target by tergetcli based on the interactive mode.
It is the sequences of the target configuraion as follows:

* Create backend storage object.
* Define the IQN(iSCSI Qualified Name).
* Define the Portal meta data.(Listen IP and port)
* Specify the ACL.

The targetcli have the virtual directory tree that have the target components by hierarchy.
The 'ls' command in the targetcli can list the configurations data at that layers such like bash 'ls' command.

It's the inital output that have no configurations about the target as follows:

```bash
# targetcli
targetcli shell version 2.1.fb41
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / ............................................................ [...]
  o- backstores ................................................. [...]
  | o- block ..................................... [Storage Objects: 0]
  | o- fileio .................................... [Storage Objects: 0]
  | o- pscsi ..................................... [Storage Objects: 0]
  | o- ramdisk ................................... [Storage Objects: 0]
  o- iscsi ............................................... [Targets: 0]
  o- loopback ............................................ [Targets: 0]
/>
```

Make the block backend stores with one disk storage and one image file.

```bash
/> cd backstores/block
/backstores/block> ls
o- block ......................................... [Storage Objects: 0]
/backstores/block> create name=diskblock1 dev=/dev/vdb1
Created block storage object diskblock1 using /dev/vdb1.
/backstores/block> cd ../fileio
/backstores/fileio> create name=fileblock1 file_or_dev=/fimg/1gbfile.img
Created fileio fileblock1 with size 1073741824
/backstores/block> cd /
/> ls
o- / ................................................................ [...]
  o- backstores ..................................................... [...]
  | o- block ......................................... [Storage Objects: 1]
  | | o- diskblock1 ........ [/dev/vdb1 (1023.0MiB) write-thru deactivated]
  | o- fileio ........................................ [Storage Objects: 1]
  | | o- fileblock1 ... [/fimg/1gbfile.img (1.0GiB) write-back deactivated]
  | o- pscsi ......................................... [Storage Objects: 0]
  | o- ramdisk ....................................... [Storage Objects: 0]
  o- iscsi ................................................... [Targets: 0]
  o- loopback ................................................ [Targets: 0]
/>
```

Subsequently, define the IQN. (IQN naming convention is in the RFC 3720.)

```bash
/> cd iscsi
/iscsi> create iqn.2016-12.local.host.target:dskarray1
Created target iqn.2016-12.local.host.target:dskarray1.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
/iscsi> ls /
o- / ................................................................ [...]
  o- backstores ..................................................... [...]
  | o- block ......................................... [Storage Objects: 1]
  | | o- diskblock1 ........ [/dev/vdb1 (1023.0MiB) write-thru deactivated]
  | o- fileio ........................................ [Storage Objects: 1]
  | | o- fileblock1 ... [/fimg/1gbfile.img (1.0GiB) write-back deactivated]
  | o- pscsi ......................................... [Storage Objects: 0]
  | o- ramdisk ....................................... [Storage Objects: 0]
  o- iscsi ................................................... [Targets: 1]
  | o- iqn.2016-12.local.host.target:dskarray1 .................. [TPGs: 1]
  |   o- tpg1 ...................................... [no-gen-acls, no-auth]
  |     o- acls ................................................. [ACLs: 0]
  |     o- luns ................................................. [LUNs: 0]
  |     o- portals ........................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ............................................ [OK]
  o- loopback ................................................ [Targets: 0]
/iscsi>
```

The portal was created by default value '0.0.0.0:3260' when the IQN defined.
But we'll change the specific IP '192.168.1.28'.

```bash
/iscsi> cd iqn.2016-12.local.host.target:dskarray1/tpg1/portals/
/iscsi/iqn.20.../tpg1/portals> delete 0.0.0.0 ip_port=3260
Deleted network portal 0.0.0.0:3260
/iscsi/iqn.20.../tpg1/portals> create 192.168.1.28 ip_port=3260
Using default IP port 3260
Created network portal 192.168.1.28:3260.
/iscsi/iqn.20.../tpg1/portals> ls /
o- / ................................................................ [...]
  o- backstores ..................................................... [...]
  | o- block ......................................... [Storage Objects: 1]
  | | o- diskblock1 ........ [/dev/vdb1 (1023.0MiB) write-thru deactivated]
  | o- fileio ........................................ [Storage Objects: 1]
  | | o- fileblock1 ... [/fimg/1gbfile.img (1.0GiB) write-back deactivated]
  | o- pscsi ......................................... [Storage Objects: 0]
  | o- ramdisk ....................................... [Storage Objects: 0]
  o- iscsi ................................................... [Targets: 1]
  | o- iqn.2016-12.local.host.target:dskarray1 .................. [TPGs: 1]
  |   o- tpg1 ...................................... [no-gen-acls, no-auth]
  |     o- acls ................................................. [ACLs: 0]
  |     o- luns ................................................. [LUNs: 0]
  |     o- portals ........................................... [Portals: 1]
  |       o- 192.168.1.28:3260 ....................................... [OK]
  o- loopback ................................................ [Targets: 0]
/iscsi/iqn.20.../tpg1/portals>
```

It's the configuration to make the backend stores establish to LUNs into the IQN.

```bash
/iscsi/iqn.20.../tpg1/portals> cd ../luns
/iscsi/iqn.20...ay1/tpg1/luns> create /backstores/block/diskblock1
Created LUN 0.
/iscsi/iqn.20...ay1/tpg1/luns> create /backstores/fileio/fileblock1
Created LUN 1.
/iscsi/iqn.20...ay1/tpg1/luns> ls /
o- / ................................................................ [...]
  o- backstores ..................................................... [...]
  | o- block ......................................... [Storage Objects: 1]
  | | o- diskblock1 .......... [/dev/vdb1 (1023.0MiB) write-thru activated]
  | o- fileio ........................................ [Storage Objects: 1]
  | | o- fileblock1 ..... [/fimg/1gbfile.img (1.0GiB) write-back activated]
  | o- pscsi ......................................... [Storage Objects: 0]
  | o- ramdisk ....................................... [Storage Objects: 0]
  o- iscsi ................................................... [Targets: 1]
  | o- iqn.2016-12.local.host.target:dskarray1 .................. [TPGs: 1]
  |   o- tpg1 ...................................... [no-gen-acls, no-auth]
  |     o- acls ................................................. [ACLs: 0]
  |     o- luns ................................................. [LUNs: 2]
  |     | o- lun0 .......................... [block/diskblock1 (/dev/vdb1)]
  |     | o- lun1 ................. [fileio/fileblock1 (/fimg/1gbfile.img)]
  |     o- portals ........................................... [Portals: 1]
  |       o- 192.168.1.28:3260 ....................................... [OK]
  o- loopback ................................................ [Targets: 0]
/iscsi/iqn.20...ay1/tpg1/luns>
```

It need the ACL configurations if you need access control over the allowed clients.
It is based on the client initiator name to identify the clients whether allowed or not from the target.
We assume the allowed initiator name is 'iqn.2016-12.local.host.initiator:client1' that will be configured later at the initiator configurations part.

```bash
/iscsi/iqn.20...ay1/tpg1/luns> cd ../acls
/iscsi/iqn.20...ay1/tpg1/acls> create iqn.2016-12.local.host.initiator:client1
Created Node ACL for iqn.2016-12.local.host.initiator:client1
Created mapped LUN 1.
Created mapped LUN 0.
/iscsi/iqn.20...ay1/tpg1/acls> ls /
o- / ................................................................ [...]
  o- backstores ..................................................... [...]
  | o- block ......................................... [Storage Objects: 1]
  | | o- diskblock1 .......... [/dev/vdb1 (1023.0MiB) write-thru activated]
  | o- fileio ........................................ [Storage Objects: 1]
  | | o- fileblock1 ..... [/fimg/1gbfile.img (1.0GiB) write-back activated]
  | o- pscsi ......................................... [Storage Objects: 0]
  | o- ramdisk ....................................... [Storage Objects: 0]
  o- iscsi ................................................... [Targets: 1]
  | o- iqn.2016-12.local.host.target:dskarray1 .................. [TPGs: 1]
  |   o- tpg1 ...................................... [no-gen-acls, no-auth]
  |     o- acls ................................................. [ACLs: 1]
  |     | o- iqn.2016-12.local.host.initiator:client1 .... [Mapped LUNs: 2]
  |     |   o- mapped_lun0 ................... [lun0 block/diskblock1 (rw)]
  |     |   o- mapped_lun1 .................. [lun1 fileio/fileblock1 (rw)]
  |     o- luns ................................................. [LUNs: 2]
  |     | o- lun0 .......................... [block/diskblock1 (/dev/vdb1)]
  |     | o- lun1 ................. [fileio/fileblock1 (/fimg/1gbfile.img)]
  |     o- portals ........................................... [Portals: 1]
  |       o- 192.168.1.28:3260 ....................................... [OK]
  o- loopback ................................................ [Targets: 0]
/iscsi/iqn.20...ay1/tpg1/acls>
```

Finally, the target configuration is complete after saving the configurations, but the auto save is default setting.

```bash
/iscsi/iqn.20...ay1/tpg1/acls> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup.
Configuration saved to /etc/target/saveconfig.json
```
```bash
# targetcli saveconfig
Last 10 configs saved in /etc/target/backup.
Configuration saved to /etc/target/saveconfig.json
```

#### Step3: The prerequisite tasks for the initiator configuration.
The initiator means the client for the iSCSI target.
It's same as the configurations as of RHEL6/CentOS6 unlike the target configurations.

The 'iscsid.service' processes the iSCSI command from the backend, and 'iscsi.service' recognizes automatically the registration of the target when the system start.

```bash
# systemctl is-enabled iscsi
enabled

# systemctl is-enabled iscsid
disabled

# systemctl enable iscsid
Created symlink from /etc/systemd/system/multi-user.target.wants/iscsid.service to /usr/lib/systemd/system/iscsid.service.

# systemctl start iscsid
```

I said the initiator is named the 'iqn.2016-12.local.host.initiator:client1' at the previous ACL configuration of the target part.

```bash
# vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2016-12.local.host.initiator:client1

# systemctl restart iscsid
```

#### Step4: The main initiator configurations.
This initiator configurations are based on the iscsiadm command as follows.

I need to discovery the target to identify the IQN, the node mode is the setting for the specific IQN.

```bash
# iscsiadm --mode discovery --type sendtargets --portal 192.168.1.28:3260
192.168.1.28:3260,1 iqn.2016-12.local.host.target:dskarray1

# iscsiadm --mode node --targetname iqn.2016-12.local.host.target:dskarray1 --portal 192.168.1.28:3260 --login
Logging in to [iface: default, target: iqn.2016-12.local.host.target:dskarray1, portal: 192.168.1.28,3260] (multiple)
Login to [iface: default, target: iqn.2016-12.local.host.target:dskarray1, portal: 192.168.1.28,3260] successful.
```

Check the iscsi session status.

```bash
# iscsiadm --mode session -P 3

iSCSI Transport Class version 2.0-870
version 6.2.0.873-33.2
Target: iqn.2016-12.local.host.target:dskarray1 (non-flash)
        Current Portal: 192.168.1.28:3260,1
        Persistent Portal: 192.168.1.28:3260,1
                **********
                Interface:
                **********
                Iface Name: default
                Iface Transport: tcp
                Iface Initiatorname: iqn.2016-12.local.host.initiator:client1
 ...snip...
                ************************
                Attached SCSI devices:
                ************************
                Host Number: 5  State: running
                scsi5 Channel 00 Id 0 Lun: 0
                        Attached scsi disk sda          State: running
                scsi5 Channel 00 Id 0 Lun: 1
                        Attached scsi disk sdb          State: running
```

Done.
