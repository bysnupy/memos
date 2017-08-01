## Practical notes for OpenStack installation with packstack

* Installing packstack with yum
```bash
# yum install openstack-packstack
```

* Generating the answer file
```bash
# packstack --gen-answer-file /path/to/answer.file
```

* Edit the answer file
```bash
CONFIG_KEYSTONE_ADMIN_PW=adminpw
...snip...
```

* Installing the OpenStack with packstack

```bash
# packstack --answer-file /path/to/answer.file
...snip...
```
* Reexecuting with modified answer files

```bash
-- If you want to install nova after installation with CONFIG_NOVA_INSTALL=n, you just modified the answerfile and reexecute packstack
# vim /path/to/answer.file
CONFIG_NOVA_INSTALL=y
...snip...

# packstack --answer-file /path/to/answer.file
```
