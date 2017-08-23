## What is an Object Storage ?

### Introduction

Describing about an object storage architecture.

### Terms

* Object

An object is the stored data in the Object Storage.

* Object Container

A container stores objects like directory except not to be nested it.

* Pseudo folder

A pseudo folder is a hierarchical structure for organizing objects in the container.

### Object Storage

Store and retrieve datas without a file system interface.

#### For example, comparison between Swift and Ceph

Field | Swift | Ceph
-|-|-
Consistency model	| Eventual consistency |Strong consistency
Access method |	RESTful API |	RESTful API
Object Storage support | 	O	 | O
Block Storage support |	X | O
File Based Storage support | X | O
