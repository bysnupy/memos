## Red Hat Cluster add-on components comparison

### Introduction

The comparison memo for difference of RHEL HA add-on between 6 and 7 major version.

### Comparison

Feature|rgmanager|Pacemaker
-|-|-
Resource Configuration Management|Manual|Automatic
Resource Management Model|Resource Group|Resource-Dependency, Resource Group
Dependency Models|Colocation, Start-After|User-defined
Event Handling Model|Distributed or Centralized|Centralized
Command-Line Interface Management|Status, Control|Status, Control, Administration
Fencing Model|Assumed|Flexible
Multi-State Resources|No|Yes
Event Scripts|Yes|No
Maximum Node Count|16|16
Exclusive Services|Yes|Yes
Failover Domains|Yes|Yes
Resource Exclusion|No|Yes
Time-Based Resource Control|No|Yes
Resource Attribute Inheritance|Yes|Yes
Shared Resources|Yes|Yes
Cloned Resources|No|Yes
Resource Agent APIs|OCF, SysV|OCF, SysV
Resource Freezing|Yes|Yes
Requires Quorum|Yes|Configurable
Requires DLM|Yes|No
Multi-Partition Resource Management|No|Yes
Non-root Administration|No|Yes

### Feature Descriptions

#### Resource Configuration Management

* Manual
Changes to the resource configuration in 'cluster.conf' must be manually synchronized with the other nodes in the cluster using the ccs command.

* Automatic
Changes to the resource configuration can be made on any cluster node and Pacemaker will automatically synchronize them to active peers.
The configuration is also versioned, so any node that was not active when the change(s) were made will receive them when it starts.

#### Resource Management Model

* Resource Group: In rgmanager, resources are managed as part of a single group, with all operations performed on them in a defined order.

* Resource-Dependency: In Pacemaker, resources are managed individually, but can be configured to run on the same node as others, or "follow" them, through colocation constraints.

#### Dependency Models

* Colocation: Place a resource or resource group relative to a dependency

* Start-After: Only start a resource or resource group after a dependency has been started

* User-Defined: Use colocation, ordering, and location constraints to define a custom policy per-resource.

#### Event Handling Model

* Distributed: All nodes in the cluster determine the appropriate action to take based on the state of resources/groups, current membership, and events.

* Centralized: A single node determines the appropriate actions and issues instructions to other nodes to carry out

#### Command-Line Interface Management

* Status: Determine the state of resources or groups

* Control: Change the state of resources or groups, such as starting, stopping, relocating, etc.

* Administration: Configure resource attributes and layout

#### Fencing Model

* Assumed: The resource manager waits for the cluster infrastructure to acknowledge that fencing of a node has completed before resuming operation and initiating recovery of its resources.

* Flexible:
Only resources that require fencing will be blocked by Pacemaker until fencing has completed. Resources known to not be running on a failed node may also continue to be managed by the resource manager without waiting for fencing. Fencing is also used to allow recovery when resources can or will not clean up after themselves.

#### Event Scripts

rgmanager offers a mechanism to script event-based actions using RIND and S/Lang. It requires custom script development and can be complicated. Pacemaker offers no such feature but can provide alerts when a resource changes state.

#### Maximum Node Count

In RHEL 6, Red Hat supports pacemaker clusters of up to 16 nodes

#### Exclusive Services

The ability to configure resources or resource groups to run only on a node that has no other resources or groups running. In rgmanager, this is done using its central_processing mode and service depend and depend_mode attributes. With Pacemaker, exclusivity can be accomplished through resource dependencies.

#### Failover Domains

Defined subsets of nodes that are eligible to run a resource or group, and the ordering in which they are preferred, along with options for controlling the failover behavior.

#### Resource Exclusion

##### Time-Based Resource Control

With Pacemaker, admins can configure the cluster to behave differently based on the current date/time. For example, waiting until non-business hours to move a service if it's primary host becomes available after a failure, or running more copies of a web server during the day than at night.

##### Resource Attribute Inheritance

The ability of a resource to inherit a specific attribute value from an associated resource. In rgmanager, this is done through parent-child relationships within a service (for example, an nfsclient resource inherting the path to export from its parent fs resource's mountpoint attribute).

#### Shared Resources

The ability for a resource to be utilized by multiple groups.

#### Cloned Resources

Pacemaker offers the ability to run the same resource, a web server or cluster file system for example, on multiple nodes.

#### Multi-State Resources

Pacemaker offers the ability to run the same resource on multiple nodes but with different modes, such as master and slave.
What the modes mean can be specific to your application, but a common usage is for them to map to targets and sources for synchronizing data.

#### Resource Agent APIs

* OCF: The Open Cluster Framework standard that is an extension of LSB. rgmanager does not use the monitor operation defined in OCF, but rather uses status.

* SysV: The format used by runlevel-based init scripts

#### Resource Freezing

The ability to temporarily halt status checks for a given resource, and thereby prevent any recovery action should the resource fail. Often useful for maintenance operations during which a resource's state may change but administrators wish to avoid a recovery event.

#### Requires Quorum

Quorum is a minimum number of votes that must be present in a cluster in order for certain operations to succeed. By default, quorum is greater than half of the total number of votes configured in the cluster. rgmanager is unable to manage resources or provide high availability if quorum is lost, whereas Pacemaker allows configurable behavior with regards to quorum on a per-resource basis.

#### Requires DLM

rgmanager depends on the Distributed Lock Manager to maintain consistent states across the cluster. Pacemaker does not.

#### Multi-Partition Resource Management

A cluster partition is a group of nodes that can communicate with each other but no-one else.
When a cluster splits into two or more partitions, in the partition(s) without quorum, rgmanager will freeze waiting to be fenced or for the membership to change.

In contrast, Pacemaker's behavior is configurable, it can:

* Pretend it still has quorum, try to fence the other partition and recover its resources (useful for 2 node clusters)

* Continue managing the resources in the partition but not attempt to recover resources from the other one(s)

* Begin shutting down active services and wait for fencing to occur or the partitions to merge

* Fence all members of its own partition

#### Non-root Administration

The ability to control and configure resources as a regular, non-root user.
