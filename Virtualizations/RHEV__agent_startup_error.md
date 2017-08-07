## ovirt-guest-agent daemon doesn't start

Environment

Item | Value
-|-
OS | RHEL6 or CentOS6
udev | < 147-2.73.el6_8.2

#### Symptom

* Error messages, /var/log/messages

```
MainThread::ERROR::2017-08-07 14:10:37,654::ovirt-guest-agent::103::root::Unhandled exception in oVirt guest agent!
Traceback (most recent call last):
  File "/usr/share/ovirt-guest-agent/ovirt-guest-agent.py", line 97, in <module>
    agent.run(daemon, pidfile)
  File "/usr/share/ovirt-guest-agent/ovirt-guest-agent.py", line 30, in run
    self.agent = LinuxVdsAgent(config)
  File "/usr/share/ovirt-guest-agent/GuestAgentLinux2.py", line 201, in __init__
    AgentLogicBase.__init__(self, config)
  File "/usr/share/ovirt-guest-agent/OVirtAgentLogic.py", line 55, in __init__
    self.vio = VirtIoChannel(config.get("virtio", "device"))
  File "/usr/share/ovirt-guest-agent/VirtIoChannel.py", line 87, in __init__
    self._vport = os.open(vport_name, os.O_RDWR)
OSError: [Errno 13] Permission denied: '/dev/virtio-ports/com.redhat.rhevm.vdsm'
```

Root cause is udev bug

#### Solutions

* upgrade udev package more than 147-2.73.el6_8.2, and then reinstall rhevm-guest-agent

* temporal workaround as following commands

```
# /sbin/udevadm trigger --subsystem-match="virtio-ports" --attr-match="name=com.redhat.rhevm.vdsm"
# service ovirt-guest-agent start
```
