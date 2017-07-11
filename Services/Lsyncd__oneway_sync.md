## One-way synchronization (lsyncd + rsync)
### Introduction
The [lsyncd](https://github.com/axkibe/lsyncd) + rsync combination stack is efficient and simple for the small-scale synchronizing systems.
rsync combination stack is efficient and simple for the small-scale synchronizing systems.
For instance, the irregularly uploaded image files would be synchronied between one source host between some target hosts,
if your system doesn't strictly depend on sync speed.

This quick reference is the memo that I had simulated the one-way sync configuration with lsyncd as non-root user.

### Environment
* Host: tgt1.host.local(tgt1), tgt2.host.local(tgt2), org.host.local(org)
* OS: CentOS 7 (All the hosts are same.)
* Packages: lsyncd-2.1.5-6.el7, rsync-3.0.9-17.el7

All hosts are required the rsync package but the lsyncd package is installed at org.host.local only.

### Configuration Summary
The webmgr user have just symbolic links that linked the contents of the other users: imgcms, imgcgi.
But the contents on the org.host.local will be synchronized to be real contents of webmgr on the tgt1,2.host.local, not be symbolic links.
![lsyncd oneway sync img1](https://github.com/bysnupy/memos/blob/draft/Services/images/lsyncd__oneway_diagram1.png)
