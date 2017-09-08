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

* Primary OSD functions:<br/>
.Serves all I/O requests<br/>
.Responsible for replication<br/>
.Responsible for coherency<br/>
.Responsible for rebalancing<br/>
.Responsible for recovery<br/>

* Secondary OSD functions:<br/>
.Under control of the primary<br/>
.Capable of becoming primary<br/>

### Ceph access methods

* Ceph native API(librados)
* Ceph Gateway(RADOSGW), RESTful API
* Ceph Block Device(RBD, librbd)
* Ceph File system(CephFS, libcephfs)





