## What is IPv4 networking?

### Itroduction

TCP/IP standards have 4 layers network models (RFC1122)

* Application layer

To communicate with each other, applications have the specifications.
Such as, SSH to authenticate to remote server, HTTP provides secure web access, SMTP is email delivery service, NFS and CIFS provides file sharing and FTP is file transfer service.

* Transport layer

TCP is connection oriented communication protocol, UDP is connectionless datagram protocol.
Applications use these protocols by exchanging packets (TCP and UDP)

* Network layer or Internet layer

IP protocol carry the data from source host to destination host with IP address.
IP address prefix be used to determine network address, routers use this network to connect multiple such networks.
ICMP protocol is using packet types to control some tasks like ping.

* Link layer or media access layer

This layer provides the connection by using physical device and media, such as ethernet and wireless WLAN.
These types connect to destination using MAC addresses on the Local networks. (MAC address is used to identity destination of packets on the local networks)


* IP address and netmask

IP address is made up 32 bit numbers (8bit x 4). The network and host addresses be identified by netmask.
All hosts can communicate with same subnet networks witout a router. (same network part, but not same host part)
Lowest address is network address, highest address is broadcast address.





