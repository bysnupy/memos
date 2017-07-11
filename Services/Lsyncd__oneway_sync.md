## One-way synchronization (lsyncd + rsync)
### Introduction
The [lsyncd](https://github.com/axkibe/lsyncd) + rsync combination stack is efficient and simple for the small-scale synchronizing systems.
rsync combination stack is efficient and simple for the small-scale synchronizing systems.
For instance, the irregularly uploaded image files would be synchronied between one source host between some target hosts,
if your system doesn't strictly depend on sync speed.

This quick reference is the memo that I had simulated the one-way sync configuration with lsyncd as non-root user.

### Environment
Item|Value
--|--
Host| tgt1.host.local(tgt1), tgt2.host.local(tgt2), org.host.local(org)
OS| CentOS 7 (All the hosts are same.)
Packages| lsyncd-2.1.5-6.el7, rsync-3.0.9-17.el7

All hosts are required the rsync package but the lsyncd package is installed at org.host.local only.

### Configuration Summary
The webmgr user have just symbolic links that linked the contents of the other users: imgcms, imgcgi.
But the contents on the org.host.local will be synchronized to be real contents of webmgr on the tgt1,2.host.local, not be symbolic links.
![lsyncd oneway sync img1](https://github.com/bysnupy/memos/blob/draft/Services/images/lsyncd__oneway_diagram1.png)

### Configuration steps

#### Step1: Creating the test users and change the access permission of the home directories.

We need the test users and directory trees as follows.

This webmgr user is accessible to the contents of the imgcms, cgicms users through the symbolic links in the org.host.local host.
But the remote webmgr user will be synchronized as its own permission in the tgt1,tgt2.host.local hosts.

```bash
org.host.local : uid=12071(webmgr) gid=12071(webmgr) groups=12071(webmgr),12069(imgcms),12070(cgicms)
tgt1.host.local, tgt2.host.local : uid=12071(webmgr) gid=12071(webmgr) groups=12071(webmgr)
```

We assume that the users as follows are the owners of the contents in the org.host.local host.

```bash
org.host.local : uid=12069(imgcms) gid=12069(imgcms) groups=12069(imgcms)
org.host.local : uid=12070(cgicms) gid=12070(cgicms) groups=12070(cgicms)
```

The directory tree of the org.host.local host.

```bash
/home
├── [drwx------ webmgr.webmgr]  webmgr
│     └── [drwxrwxr-x webmgr.webmgr]  contents
│            ├── [lrwxrwxrwx webmgr.webmgr]  images -> /home/imgcms/images
│            └── [lrwxrwxrwx webmgr.webmgr]  cgi-bins -> /home/cgicms/cgi-bins
├── [drwx--x--- imgcms.imgcms]  imgcms
│     └── [drwxrwxr-x imgcms.imgcms]  images
└── [drwx--x--- cgicms.cgicms]  cgicms
       └── [drwxrwxr-x cgicms.cgicms]  cgi-bins
```

The directory tree of the tgt1.host.local and tgt2.host.local hosts.

```bash
/home
└── [drwx------ webmgr.webmgr]  webmgr
       └── [drwxrwxr-x webmgr.webmgr  ]  contents
              ├── [drwxrwxr-x webmgr.webmgr]  images
              └── [drwxrwxr-x webmgr.webmgr]  cgi-bins
```

You should set up the password to the users with passwd command after creating the users as follows.

```bash
[org ]# useradd imgcms
[org ]# useradd cgicms
[org ]# useradd -G imgcms,cgicms webmgr
[org ]# setfacl -m u:webmgr:x /home/imgcms
[org ]# setfacl -m u:webmgr:x /home/cgicms

[tgt1 ]# useradd webmgr

[tgt2 ]# useradd webmgr
```

#### Step2: The Configuration of The key-based authentication through SSH.

The key-based authentication is required to the synchronization of the contents through the SSH without password.

The commands should be executed by webmgr user as follows.

```bash
[org webmgr]$ ssh-keygen -t rsa -b 2048

Generating public/private rsa key pair.
Enter file in which to save the key (/home/webmgr/.ssh/id_rsa): /home/webmgr/.ssh/sync-rsa2048.key
Created directory '/home/webmgr/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/webmgr/.ssh/sync-rsa2048.key.
Your public key has been saved in /home/webmgr/.ssh/sync-rsa2048.key.pub.
The key fingerprint is:
db:4b:18:a8:a8:0a:df:84:01:a5:c5:53:e3:88:3c:f2 webmgr@org.host.local
The key's randomart image is:
+--[ RSA 2048]----+
| .o.o            |
|.=oo .           |
|=o...            |
|.o.    .         |
|  E   . S        |
|   + .   =       |
|. o o   o o      |
|.o o     . .     |
|+ . .     .      |
+-----------------+
```

The transportation of the public key to the remote hosts.

```bash
[org webmgr]$ ssh-copy-id -i ~/.ssh/sync-rsa2048.key.pub webmgr@tgt1.host.local

/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
webmgr@tgt1.host.local's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'webmgr@tgt1.host.local'"
and check to make sure that only the key(s) you wanted were added.

[org webmgr]$ ssh-copy-id -i ~/.ssh/sync-rsa2048.key.pub webmgr@tgt2.host.local

/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys

webmgr@tgt2.host.local's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'webmgr@tgt2.host.local'"
and check to make sure that only the key(s) you wanted were added.
Test the key-based authentication through the SSH without password.
You should type "yes" to accept the authenticity of the remote host at the first time connection.

[org webmgr]$ ssh -i ~/.ssh/sync-rsa2048.key tgt1.host.local
... snip ...
[tgt1 webmgr]$ hostname -f
tgt1.host.local 

[org webmgr]$ ssh -i ~/.ssh/sync-rsa2048.key tgt2.host.local
... snip ...

[tgt2 webmgr]$ hostname -f
tgt2.host.local
```

#### Step3: Configuration of the lsyncd.conf file.

I process the multiple targets list with lua loops in here (reference link), because you can use the lua into the 'lsyncd.conf' file. 
And the variables that have prefix 'lcv_' are not the directives of the lsyncd, just temporarily defined local variables.
The rsync block options(archive, compress) are same as '-az' options of rsync command. 

```bash
[org ]# vim /etc/lsyncd.conf

--------
---- Description: the Lsyncd main configuration file - one-way
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
-- lcv_rsh = "/usr/bin/ssh -i /home/webmgr/.ssh/sync-rsa2048.key -o StrictHostKeyChecking=no"
lcv_rsh = "/usr/bin/ssh -i /home/webmgr/.ssh/sync-rsa2048.key"

-- The image contents target list local variable.
lcv_tgtlist_imgcms = {
  "webmgr@tgt1.host.local:/home/webmgr/contents/images",
  "webmgr@tgt2.host.local:/home/webmgr/contents/images"
}

-- The cgi contents target list local variable.
lcv_tgtlist_cgicms = {
  "webmgr@tgt1.host.local:/home/webmgr/contents/cgi-bins",
  "webmgr@tgt2.host.local:/home/webmgr/contents/cgi-bins"
}

----
-- The main loops
--

-- The sync block of the image contents.
for _, lcv_tgt in ipairs( lcv_tgtlist_imgcms ) do
  sync {
    default.rsync,
    source="/home/imgcms/images/",
    target=lcv_tgt,
    rsync = {
      archive = true,
      compress = true,
      rsh = lcv_rsh,
    }
  }
end

-- The sync block of the cgi contents.
for _, lcv_tgt in ipairs( lcv_tgtlist_cgicms ) do
  sync {
    default.rsync,
    source="/home/cgicms/cgi-bins/",
    target=lcv_tgt,
    rsync = {
      archive = true,
      compress = true,
      rsh = lcv_rsh,
    }
  }
end
```

#### Step4: Start the lsyncd and test the configurations.

You need to make the log directory: /var/log/lsyncd, if you start the lsyncd daemon initially.

```bash
[org ]# mkdir /var/log/lsyncd
[org ]# systemctl start lsyncd
```

There are tests for creating the file, changing the contents, changing the access mode and renaming under the symbolic link.

```bash
[org webmgr]$ touch ~/contents/images/owneriswebmgrundersymlinks.txt
[org webmgr]$ echo "Watch that the file size is changing" > ~/contents/images/owneriswebmgrundersymlinks.txt
[org webmgr]$ chmod 600 ~/contents/images/owneriswebmgrundersymlinks.txt
[org webmgr]$ mv ~/contents/images/owneriswebmgrundersymlinks.txt ~/contents/images/webmgrthroughsymlnk.txt
```

These tests are deleting files.

```bash
[org cgicms]$ touch ~/cgi-bins/cgi{1..9}.cgi
[org cgicms]$ rm -f ~/cgi-bins/cgi{5..9}.cgi
```

You should monitor the target directories in the remote hosts (tgt1, tgt2), while the tests are performing.
For example, on the tgt1 hosts as follows.

```bash
[tgt1 wegmgr]$ watch -d -n1 ls -lR ~/contents
Every 1.0s: ls -lR /home/webmgr/contents             Tue Nov 29 18:29:41 2016

/home/webmgr/contents:
total 0
drwxrwxr-x. 2 webmgr webmgr 66 Nov 29 18:29 cgi-bins
drwxrwxr-x. 2 webmgr webmgr 36 Nov 29 18:28 images

/home/webmgr/contents/cgi-bins:
total 0
-rw-rw-r--. 1 webmgr webmgr 0 Nov 29 18:27 cgi1.cgi
-rw-rw-r--. 1 webmgr webmgr 0 Nov 29 18:27 cgi2.cgi
-rw-rw-r--. 1 webmgr webmgr 0 Nov 29 18:27 cgi3.cgi
-rw-rw-r--. 1 webmgr webmgr 0 Nov 29 18:27 cgi4.cgi

/home/webmgr/contents/images:
total 4
-rw-------. 1 webmgr webmgr 37 Nov 29 18:24 webmgrthroughsymlnk.txt
```
