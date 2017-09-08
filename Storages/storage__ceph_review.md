## Ceph Storage review

### Intorduction

Ceph is a unified storage platform that supports object, file and block storage.


### Access methods of Ceph storage

* Block storage integrated with OpenStack, Linux, KVM hypervisor
* Object storage access through AWS S3, Swift
* Disaster recovery and multisite options

### Technological features

* scalable for every component
* No single point of failure
* Software-based and open source
* self-managed

* CRUSH which aware physical topology of infrastructures
* No need for location metadata
* Parallelism for enhanced throughput
* Reliable Autonomic Distributed Object Store (RADOS)

### Ceph components

Component | Description
-|-
Ceph monitors | monitoring cluster states
Ceph Object Storage Devices | storage building blocks, support only XFS(with xattrs, extended attributes)                             
Metadata server | metadata management which supports directory hierarchy and file metadata

* Primary OSD functions:
.Serves all I/O requests
.Responsible for replication
.Responsible for coherency
.Responsible for rebalancing
.Responsible for recovery

* Secondary OSD functions:
.Under control of the primary
.Capable of becoming primary

