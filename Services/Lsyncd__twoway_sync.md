## Two-way bidirectional synchronization (lsyncd + rsync)
### Introduction
One day a project manager had consulted me about the file share solutions which are good cost-performance between 2 web servers under the Load balancer. The project manager said that the NFS or DRBD is overkill to that system and having the budget problem, so he wanted to implement the file share with the Lsyncd + rsync. But the Lsyncd didn't support the bidirectional synchronizaion and it's difficult to implement the complete bidirectional synchronization according to the limited architecture which didn't have the mechanism for concurrency control like the DBMS.
The file synchronization with Lsync + rsync seems to work well in common conditions but it will be the critical problems if either of the 2 web servers have the problems; such as network or OS troubles. No servers can know which data condition is right, and can't control over the synchronization as one transaction across the two servers. So we would need the rules to comply with the file synchronization principles between 2 web servers if either of the 2 web servers make problems.

![trouble case](https://github.com/bysnupy/memos/blob/draft/Services/images/lsyncd__twoway_diagram1.png)

:exclamation: In advance, I defined that this post is just written as the provision of information, I won't be not responsible for data-loss caused by this post.</span>

### Environment
Item|Value
-|-
Host|tgt-a.host.local(tgt-a), tgt-b.host.local(tgt-b)
OS|CentOS 7 (All the hosts are same.)
Packages|lsyncd-2.1.5-6.el7, rsync-3.0.9-17.el7

### Configuration Summary
We assume that the required configurations have been completed such as key-based authentication, directories tree and test users.
This post is helpful to you: [related post](https://github.com/bysnupy/memos/blob/master/Services/Lsyncd__oneway_sync.md)

You would test the configuraion with non-critical data or must do the backup before test.

I recommend the other soultions (DRBD, NFS, GlusterFS and so on) about the systems follows:(in bidirectional synchronization perspective)

* Frequently updated the files as same name.
* The synchronization unit is not a file.
* The need for updating the same file from some or many sessions.
* The synchronizaion task is critical to the business systems.
* You need to synchronize the contents on the more than 2 servers.

This configuration is going well in normal conditions as long as my cases, but you need to know this configuration in details to troubleshoot if there is a malfunction in this lsyncd + rsync stack.

Simple features summary:
* No files are deleted as soon as the lsyncd daemon started.
* It is added the files with each other without deleting as soon as the lsync daemon started if the synchronization is inconsistent.
* It is updated the newer file, if there was different contents file as same name in the other server.
* It is deleted the files as soon as the other server delete the files.
* It is changed the permission of the files as soon as the other server change the permission of the ones.
* It is added the files as soon as the other server create or add the new files.

Specially you should be careful of starting the lsyncd in the inconsistent synchronizaion status.

The below flow diagram is helpful to your understanding.

![sync process](https://github.com/bysnupy/memos/blob/draft/Services/images/lsyncd__twoway_diagram2.png)

### Configuration steps

#### Step1: Creating the rsync temporaty work directories.

This directory is used by rsync process as temporary work directory which save the transmitted files until the synchronization success.

Creating the temporary directory on the 'tgt-a.host.local'.

```bash
[tgt-a webmgr]$ mkdir -p ~/tmpdir/{imgcms,cgicms}
The directory tree on the 'tgt-a.host.local'.
/home
├── [drwx------ webmgr.webmgr]  webmgr
│      └── [drwxrwxr-x webmgr.webmgr  ]  contents
│             ├── [drwxrwxr-x webmgr.webmgr]  images
│             └── [drwxrwxr-x webmgr.webmgr]  cgi-bins
└── [drwxrwxr-x webmgr.webmgr  ]  tmpdir
        ├── [drwxrwxr-x webmgr.webmgr]  imgcms
        └── [drwxrwxr-x webmgr.webmgr]  cgicms
```

Creating the temporary directory to the each remote account on the 'tgt-b.host.local'.

```bash
[tgt-b imgcms]$ mkdir  ~/tmpdir
[tgt-b cgicms]$ mkdir  ~/tmpdir
```

The directory tree on the 'tgt-b.host.local'.

```bash
/home
├── [drwx------ webmgr.webmgr]  webmgr
│     └── [drwxrwxr-x webmgr.webmgr]  contents
│            ├── [lrwxrwxrwx webmgr.webmgr]  images -> /home/imgcms/images
│            └── [lrwxrwxrwx webmgr.webmgr]  cgi-bins -> /home/cgicms/cgi-bins
├── [drwx--x--- imgcms.imgcms]  imgcms
│     ├── [drwxrwxr-x imgcms.imgcms]  images
│     └── [drwxrwxr-x imgcms.imgcms]  tmpdir
└── [drwx--x--- cgicms.cgicms]  cgicms
       ├── [drwxrwxr-x cgicms.cgicms]  cgi-bins
       └── [drwxrwxr-x cgicms.cgicms]  tmpdir
```

#### Step2: Configuration of the lsyncd.conf in the tgt-a.host.local.
The rsync block options(update, archive, compress) are same as '-uaz' options of the rsync command.(reference link: OPTIONS SUMMARY section)
The delete="running" option means that it delete no files when it start up. (reference link: Deletes section)

```bash
[tgt-a ]# vim /etc/lsyncd.conf
--------
---- Description: the Lsyncd main configuration file - two-way
---- Host: tgt-a.host.local
----

----
-- The settings for all layers.

settings {
  logfile        = "/var/log/lsyncd/lsyncd.log",
  statusFile     = "/var/log/lsyncd/lsyncd.status",
  statusInterval = 30,
  insist         = true,
  inotifyMode    = "Modify",
  delay          = 0,
  maxProcesses   = 8
}

----
-- The local variables: the prefix is 'lcv_*'.
--

-- The local variable of the 'rsh' directive in the rsync block. 
-- lcv_rsh = "/usr/bin/ssh -i /home/webmgr/.ssh/tgt-a-sync-rsa2048.key -o StrictHostKeyChecking=no"
lcv_rsh = "/usr/bin/ssh -i /home/webmgr/.ssh/tgt-a-sync-rsa2048.key"

-- The image contents target list local variable.
lcv_tgtimg = "imgcms@tgt-b.host.local:/home/imgcms/images"

-- The cgi contents target list local variable.
lcv_tgtcgi = "cgicms@tgt-b.host.local:/home/cgicms/cgi-bins"

----
-- The main blocks
--

-- The sync block of the image contents.
sync {
  default.rsync,
  source = "/home/webmgr/contents/images/",
  target = lcv_tgtimg,
  delete = "running",
  -- init = false,
  rsync  = {
    temp_dir = "/home/imgcms/tmpdir",
    update   = true,
    archive  = true,
    compress = true,
    rsh      = lcv_rsh,
  }
}


-- The sync block of the cgi contents.
sync {
  default.rsync,
  source = "/home/webmgr/contents/cgi-bins/",
  target = lcv_tgtcgi,
  delete = "running",
  -- init = false,
  rsync  = {
    temp_dir = "/home/cgicms/tmpdir",
    update   = true,
    archive  = true,
    compress = true,
    rsh      = lcv_rsh,
  }
}
```

#### Step3: Configuration of the lsyncd.conf in the tgt-b.host.local.
It takes care that the each local source directory has the different target respectively.

```bash
[tgt-b ]# vim /etc/lsyncd.conf
--------
---- Description: the Lsyncd main configuration file - two-way
---- Host: tgt-b.host.local
----

----
-- The settings for all layers.

settings {
  logfile        = "/var/log/lsyncd/lsyncd.log",
  statusFile     = "/var/log/lsyncd/lsyncd.status",
  statusInterval = 30,
  insist         = true,
  inotifyMode    = "Modify",
  delay          = 0,
  maxProcesses   = 8
}

----
-- The local variables: the prefix is 'lcv_*'.
--

-- The local variable of the 'rsh' directive in the rsync block. 
-- lcv_rsh = "/usr/bin/ssh -i /home/webmgr/.ssh/tgt-b-sync-rsa2048.key -o StrictHostKeyChecking=no"
lcv_rsh = "/usr/bin/ssh -i /home/webmgr/.ssh/tgt-b-sync-rsa2048.key"

-- The image contents target list local variable.
lcv_tgtimg = "webmgr@tgt-a.host.local:/home/webmgr/contents/images"

-- The cgi contents target list local variable.
lcv_tgtcgi = "webmgr@tgt-a.host.local:/home/webmgr/contents/cgi-bins"

----
-- The main blocks
--

-- The sync block of the image contents.
sync {
  default.rsync,
  source = "/home/imgcms/images/",
  target = lcv_tgtimg,
  delete = "running",
  -- init = false,
  rsync  = {
    temp_dir = "/home/webmgr/tmpdir/imgcms",
    update   = true,
    archive  = true,
    compress = true,
    rsh      = lcv_rsh,
  }
}


-- The sync block of the cgi contents.
sync {
  default.rsync,
  source = "/home/cgicms/cgi-bins/",
  target = lcv_tgtcgi,
  delete = "running",
  -- init = false,
  rsync  = {
    temp_dir = "/home/webmgr/tmpdir/cgicms",
    update   = true,
    archive  = true,
    compress = true,
    rsh      = lcv_rsh,
  }
}
```

#### Step4: Tests the configuration while running on the normal conditions.
You should make the log directory as follows if the inial start of the lsyncd before starting the daemon.

```bash
[tgt-a/tgt-b ]# mkdir /var/log/lsyncd
[tgt-a/tgt-b ]# systemctl start lsyncd
```
or
```bash
[tgt-a/tgt-b ]# lsyncd -nodaemon /etc/lsyncd.conf
```

You should take care of the prompt of the test commands as follows which present the hostname where the tests were performed on.
The prompt format is as follows:

```bash
[hostname username]$
```

First tests are the delete, create, update of the files.

```bash
[tgt-a webmgr]$ touch ~/contents/cgi-bins/willbecgicmsowneratremote.txt
[tgt-b imgcms]$ touch ~/images/willbewebmgrowneratremote.txt

[tgt-b imgcms]$ touch ~/images/imgfile{1..3}.png
[tgt-a webmgr]$ rm ~/contents/images/imgfile2.png
[tgt-b imgcms]$ rm ~/images/imgfile1.png

[tgt-a webmgr]$ mv ~/contents/cgi-bins/willbecgicmsowneratremote.txt ~/contents/cgi-bins/updatetestfile.cgi
[tgt-b cgicms]$ echo "this file size will be 32 bytes" > ~/cgi-bins/updatetestfile.cgi
[tgt-a webmgr]$ echo "added this data, so ths file size are 84 bytes now." >> ~/contents/cgi-bins/updatetestfile.cgi
```

The result of the tests in the 'tgt-a.host.local'.

```bash
[tgt-a webmgr]$ watch -d -n1 ls -lR /home/webmgr

Every 1.0s: ls -lR /home/webmgr          Mon Dec  5 12:58:41 2016

/home/webmgr:
合計 0
drwxrwxr-x 4 webmgr webmgr 34 11月 29 16:27 contents

/home/webmgr/contents:
合計 0
drwxrwxr-x 2 webmgr webmgr 31 12月  5 12:48 cgi-bins
drwxrwxr-x 2 webmgr webmgr 61 12月  5 12:46 images

/home/webmgr/contents/cgi-bins:
合計 4
-rw-rw-r-- 1 webmgr webmgr 84 12月  5 12:55 updatetestfile.cgi

/home/webmgr/contents/images:
合計 0
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 12:46 imgfile3.png
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 12:46 willbewebmgrowneratremote.txt
```

The result of the tests in the 'tgt-b.host.local'.

```bash
[tgt-b webmgr]$ watch -d -n1 ls -lRL /home/webmgr

Every 1.0s: ls -lRL /home/webmgr        Mon Dec  5 13:01:33 2016

/home/webmgr:
total 0
drwxrwxr-x. 2 webmgr webmgr 34 Nov 28 16:30 contents

/home/webmgr/contents:
total 0
drwxrwxr-x. 2 cgicms cgicms 31 Dec  5 12:48 cgi-bins
drwxrwxr-x. 2 imgcms imgcms 61 Dec  5 12:46 images

/home/webmgr/contents/cgi-bins:
total 4
-rw-rw-r--. 1 cgicms cgicms 84 Dec  5 12:55 updatetestfile.cgi

/home/webmgr/contents/images:
total 0
-rw-rw-r--. 1 imgcms imgcms 0 Dec  5 12:46 imgfile3.png
-rw-rw-r--. 1 imgcms imgcms 0 Dec  5 12:46 willbewebmgrowneratremote.txt
```

#### Step5: Test about starting the lsyncd daemon when the datas states were inconsistent.
This test is the replay of the situations at the Figure2 image above and the test data states are based on the Step5 subsequently.
You need to stop the lsyncd daemon on each server before test as follows.

Fisrt we should make the files inconsistent to replay the abnormal status for tests.

The changes on the 'tgt-a.host.local'.

```bash
[tgt-a webmgr]$ rm ~/contents/images/imgfile3.png
[tgt-a webmgr]$ touch ~/contents/images/tgt-a-new{1..2}.img
[tgt-a webmgr]$ echo "the file has updated before tgt-b host." >> ~/contents/cgi-bins/updatetestfile.cgi
[tgt-a webmgr]$ $ cat ~/contents/cgi-bins/updatetestfile.cgi
this file size will be 32 bytes
added this data, so ths file size are 84 bytes now.
the file has updated before tgt-b host.
[tgt-a webmgr]$ watch -d -n1 ls -lR /home/webmgr
Every 1.0s: ls -lR /home/webmgr                                    Mon Dec  5 16:15:33 2016

/home/webmgr:
合計 0
drwxrwxr-x 4 webmgr webmgr 34 11月 29 16:27 contents

/home/webmgr/contents:
合計 0
drwxrwxr-x 2 webmgr webmgr 31 12月  5 12:48 cgi-bins
drwxrwxr-x 2 webmgr webmgr 84 12月  5 16:15 images

/home/webmgr/contents/cgi-bins:
合計 4
-rw-rw-r-- 1 webmgr webmgr 124 12月  5 16:15 updatetestfile.cgi

/home/webmgr/contents/images:
合計 0
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 16:15 tgt-a-new1.img
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 16:15 tgt-a-new2.img
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 12:46 willbewebmgrowneratremote.txt
```

The changes on the 'tgt-b.host.local'.

```bash
[tgt-b imgcms]$ rm ~/images/willbewebmgrowneratremote.txt
[tgt-b imgcms]$ touch ~/images/tgt-b-new{3..4}.jpg
[tgt-b cgicms]$ echo "The changes after making changes for tgt-a." >> ~/cgi-bins/updatetestfile.cgi
[tgt-b cgicms]$ cat ~/cgi-bins/updatetestfile.cgi

this file size will be 32 bytes
added this data, so ths file size are 84 bytes now.
The changes after making changes for tgt-a.

[tgt-b webmgr]$ watch -d -n1 ls -lRL /home/webmgr

Every 1.0s: ls -lRL /home/webmgr                                          Mon Dec  5 16:23:09 2016

/home/webmgr:
total 0
drwxrwxr-x. 2 webmgr webmgr 34 Nov 28 16:30 contents

/home/webmgr/contents:
total 0
drwxrwxr-x. 2 cgicms cgicms 31 Dec  5 12:48 cgi-bins
drwxrwxr-x. 2 imgcms imgcms 67 Dec  5 16:20 images

/home/webmgr/contents/cgi-bins:
total 4
-rw-rw-r--. 1 cgicms cgicms 128 Dec  5 16:21 updatetestfile.cgi

/home/webmgr/contents/images:
total 0
-rw-rw-r--. 1 imgcms imgcms 0 Dec  5 12:46 imgfile3.png
-rw-rw-r--. 1 imgcms imgcms 0 Dec  5 16:20 tgt-b-new3.jpg
-rw-rw-r--. 1 imgcms imgcms 0 Dec  5 16:20 tgt-b-new4.jpg
```

Let's start with the lsyncd in each server, at that time the startup order of between hosts is tgt-a -> tgt-b (to test for the 'updatetestfile.cgi' file overwriting test!), 
and then check the file directories. I'll be same status as follows:

```bash
[tgt-a/tgt-b webmgr]$ watch -d -n1 ls -lRL /home/webmgr

/home/webmgr:
合計 0
drwxrwxr-x 4 webmgr webmgr 34 11月 29 16:27 contents

/home/webmgr/contents:
合計 0
drwxrwxr-x 2 webmgr webmgr  31 12月  5 12:48 cgi-bins
drwxrwxr-x 2 webmgr webmgr 145 12月  5 16:15 images

/home/webmgr/contents/cgi-bins:
合計 4
-rw-rw-r-- 1 webmgr webmgr 128 12月  5 16:21 updatetestfile.cgi

/home/webmgr/contents/images:
合計 0
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 12:46 imgfile3.png
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 16:15 tgt-a-new1.img
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 16:15 tgt-a-new2.img
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 16:20 tgt-b-new3.jpg
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 16:20 tgt-b-new4.jpg
-rw-rw-r-- 1 webmgr webmgr 0 12月  5 12:46 willbewebmgrowneratremote.txt

[tgt-a/tgt-b webmgr]$ cat ~/cgi-bins/updatetestfile.cgi

this file size will be 32 bytes
added this data, so ths file size are 84 bytes now.
The changes after making changes for tgt-a. 
```

Done.
