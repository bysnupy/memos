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
```
# packstack --answer-file /path/to/answer.file
...snip...
```
