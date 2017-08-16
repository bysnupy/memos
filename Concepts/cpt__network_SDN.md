## What is Software Defined Networking ?

### Introduction

Software-defined networking (SDN) is a networking model that allows network administrators to manage network services through the abstraction of several networking layers. SDN decouples the software that handles the traffic, called the control plane, and the underlying mechanisms that route the traffic, called the data plane. SDN enables communication between the control plane and the data plane. For example, the OpenFlow project, combined with the OpenDaylight project, provides such implementation.

SDN does not change the underlying protocols used in networking; rather, it enables the utilization of application knowledge to provision networks. Networking protocols, such as TCP/IP and Ethernet standards, rely on manual configuration by administrators for applications. They do not manage networking applications, such as their network usage, the end point requirements, or how much and how fast the data needs to be transferred. The goal of SDN is to extract knowledge of how an application is being used by the application administrator or the application's configuration data itself.


#### Logical diagrm

![SDN layers](https://github.com/bysnupy/memos/blob/master/Concepts/images/cpt__sdn_diagram1.png)
