## Virtual machine and Container

### What Is a Virtual Machine?

System virtualization allows a single computer to contain image files, each of which is treated as a self-contained computer, or virtual machine. Each virtual machine runs its own operating system, which can be different from the one on the host computer. The virtual machines run on virtual hardware, defined and emulated by the host system. These virtual machines are isolated from each other. From the perspective of each operating system, it is running on its own private hardware. They may have their own network interfaces and IP addresses, file systems, and other peripherals. Different virtual machines need not run the same operating system or version of the operating system.

Virtualization is accomplished by using a hypervisor. A hypervisor is the software that manages and supports the virtualization environment. It runs the virtual machines for each virtualized operating system, providing access to virtual CPUs, memory, disks, networking, and other peripherals while restricting the virtual machines from having direct access to the real hardware or each other. 

### What Is a Container?

A container allows an operating system process or process tree to run isolated from other processes hosted by the same operating system. Containers can quickly scale by creating new instances of an application in a repeatable fashion.

The container framework creates an execution environment in which applications do not have access to files, devices, sockets, or processes outside of the container. If the application requires access to something external of the container, a network connection must be opened, as if the container was a separate physical server.

Each container has an exclusive virtual network interface and private IP address which allows it to access the outside world. Containers provide a way to deploy an application and its dependencies as self-contained units, without interference and conflicts caused by other applications (and their dependencies) sharing the same host OS. A container only needs the subset of resources directly required by the application, such as shared libraries, user commands, and run-time services.

