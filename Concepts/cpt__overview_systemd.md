## Systemd service manager

### Introduction

The systemd service manager is resposible for controlling how the services are started and stopped.

Primary features

* logging: since mounting init ram disk, all messages were logged by systemd journal.
* dependencies: designate the explicit dependencies to each service.
* control groups: each service is identified by each control group.
* Various unit types: it can manage vairous unit types including services.

Unit Type | Description
-|-
Devices| Create and use devices.
Mounts and automounts| Mount file systems upon request or automount a file system based on a request for a file or directory within that file system.
Paths| Check the existence of files or directories or create them as needed.
Services| Start a service, which often means launching a service daemon and related components.
Slices| Divide up computer resources (such as CPU and memory) and apply them to selected units.
Snapshots| Take snapshots of the current state of the system.
Sockets| Set up sockets to allow communication paths to processes that can remain in place, even if the underlying process needs to restart.
Swaps| Create and use swap files or swap partitions.
Targets| Manage a set of services under a single unit, represented by a target name rather than a runlevel number.
Timers| Trigger actions based on a timer.

* resource management: it manages own cgroups or slice unit.

Benefit than System V init.

* never losing the initial log messages
* respawn daemons as needed
* capture the stdout/stderr messages
* kill service cleanly
* stable services management

The Traditional Runlevels mapping to Targets

Traditional runlevel |    Target name
-|-
Runlevel 0           |    runlevel0.target -> poweroff.target
Runlevel 1           |    runlevel1.target -> rescue.target
Runlevel 2           |    runlevel2.target -> multi-user.target
Runlevel 3           |    runlevel3.target -> multi-user.target
Runlevel 4           |    runlevel4.target -> multi-user.target
Runlevel 5           |    runlevel5.target -> graphical.target
Runlevel 6           |    runlevel6.target -> reboot.target

#### Boot processes

BIOS -> Boot loader (GRUB2) -> initial RAM disk and kernel -> systemd -> initialize all system services

#### systemd start services

1. default.target (e.g. linking to multi-user.target)

```
default.target == multi-user target services (/etc/systemd/system/multi-user.target.wants)
       --call--> Requires: basic.target (/usr/lib/systemd/system/basic.target)
               --call--> Requires: sysinit.target (/usr/lib/systemd/system/sysinit.target)
```
          

```ini
-- multi-user.target
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes

-- basic.target
[Unit]
Description=Basic System
Documentation=man:systemd.special(7)

Requires=sysinit.target
After=sysinit.target
Wants=sockets.target timers.target paths.target slices.target
After=sockets.target paths.target slices.target

-- sysinit.target
[Unit]
Description=System Initialization
Documentation=man:systemd.special(7)
Conflicts=emergency.service emergency.target
Wants=local-fs.target swap.target
After=local-fs.target swap.target emergency.service emergency.target
```

#### systemctl and related commands usage

Command | Description
-|-
systemctl status SERVICE ... | check the services status
systemctl stop   SERVICE ... | stop the services
systemctl start  SERVICE ... | start the services
systemctl enable SERVICE ... | enable the services at the boot time
systemctl disable SERVICE ... | disable the services at the boot time
systemctl list-dependencies SERVICE/TARGET | show the service dependencies
systemctl list-units --type service/mount/... | list the specific unit types
systemctl list-unit-files | list all units
systemd-cgtop | CPU(c), Memory(m), Task(t), Path(p), I/O(i) can be sorted by key pressing
systemd-cgls  | list cgroups recursively
journalctl | view journal logs, -k: kernel messages, -f: follow the journal messages, -u UNITNAME: specific unit messages

#### 
