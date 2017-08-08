## Host Aggregates

#### Introduction

OpenStack have Availability Zone and Host Aggregates mechanisms to provide instances on the specific hosts.

Mechnism | Usual use cases | End user visible
-|-|-
Availability Zone | Abstraction groups to be separated geographically for high availability | Yes, but you can only show the zone name, not hostname
Host Aggregates | Logical infrastructure resource groups for enhanced performance and specific administrations | No, just Administrator can control

The following steps describe the configurations for enabling Host aggregates.

#### Configuration Steps

This configurations are conducted on the node which are providing nova-scheduler service.

* Edit the /etc/nova/nova.conf
```
-- AggregateInstanceExtraSpecsFilter conflict with ComputeCapabilitiesFilter, so add it after delete the ComputeCapabilitiesFilter
enabled_filters=RetryFilter,AvailabilityZoneFilter,RamFilter,DiskFilter,ComputeFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,CoreFilter,AggregateInstanceExtraSpecsFilter
```

* Restart the nova-scheduler service
```
systemctl restart openstack-nova-scheduler
```

#### Verifying the host aggregates
:star: There is no vm on the com1 and com2 compute nodes.

```
$ source ~/admin_credentials

-- create the com2 aggregate with metadata; com2=true
$ openstack aggregate create --property com2=true com2
+-------------------+----------------------------+
| Field             | Value                      |
+-------------------+----------------------------+
| availability_zone | None                       |
| created_at        | 2017-08-08T06:11:40.000000 |
| deleted           | False                      |
| deleted_at        | None                       |
| hosts             | []                         |
| id                | 5                          |
| metadata          | {u'com2': u'true'}         |
| name              | com2                       |
| updated_at        | 2017-08-08T06:11:40.390094 |
+-------------------+----------------------------+

-- add host
$ openstack aggregate add host com2 com2.host.local
+-------------------+----------------------------+
| Field             | Value                      |
+-------------------+----------------------------+
| availability_zone | None                       |
| created_at        | 2017-08-08T06:11:40.000000 |
| deleted           | False                      |
| deleted_at        | None                       |
| hosts             | [u'com2.host.local']       |
| id                | 5                          |
| metadata          | {u'com2': u'true'}         |
| name              | com2                       |
| updated_at        | None                       |
+-------------------+----------------------------+

-- create flavor with host aggregates metadata com2
$ openstack flavor create --public --vcpus 1 --ram 256 --property aggregate_instance_extra_specs:com2=true com2.small
+----------------------------+--------------------------------------------+
| Field                      | Value                                      |
+----------------------------+--------------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                      |
| OS-FLV-EXT-DATA:ephemeral  | 0                                          |
| disk                       | 0                                          |
| id                         | 99de5a4f-27ca-474b-98bd-991dc3375ee2       |
| name                       | com2.small                                 |
| os-flavor-access:is_public | True                                       |
| properties                 | aggregate_instance_extra_specs:com2='true' |
| ram                        | 256                                        |
| rxtx_factor                | 1.0                                        |
| swap                       |                                            |
| vcpus                      | 1                                          |
+----------------------------+--------------------------------------------+

-- launching the new instances with com2.small
$ source ~/normaluser_credentials
$ openstack server create --image cirros --flavor com2.small \
                          --nic net-id=037a5da2-28c2-494b-a0ad-0af38f499fd1 --max 2 vm
+-----------------------------+---------------------------------------------------+
| Field                       | Value                                             |
+-----------------------------+---------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                            |
| OS-EXT-AZ:availability_zone |                                                   |
| OS-EXT-STS:power_state      | NOSTATE                                           |
| OS-EXT-STS:task_state       | scheduling                                        |
| OS-EXT-STS:vm_state         | building                                          |
| OS-SRV-USG:launched_at      | None                                              |
| OS-SRV-USG:terminated_at    | None                                              |
| accessIPv4                  |                                                   |
| accessIPv6                  |                                                   |
| addresses                   |                                                   |
| adminPass                   | yCnsheDCMZn8                                      |
| config_drive                |                                                   |
| created                     | 2017-08-08T06:23:23Z                              |
| flavor                      | com2.small (99de5a4f-27ca-474b-98bd-991dc3375ee2) |
| hostId                      |                                                   |
| id                          | 10332083-ae06-420e-846f-8a8df2fe0338              |
| image                       | cirros (02815df0-a668-4e37-ba8f-63359d97986b)     |
| key_name                    | None                                              |
| name                        | vm-1                                              |
| progress                    | 0                                                 |
| project_id                  | eb77dadcd5a14f93ae89fa5e46e48e40                  |
| properties                  |                                                   |
| security_groups             | name='default'                                    |
| status                      | BUILD                                             |
| updated                     | 2017-08-08T06:23:23Z                              |
| user_id                     | 3205d8ec2cce4ed9b370d5b86357f61d                  |
| volumes_attached            |                                                   |
+-----------------------------+---------------------------------------------------+

-- list the instances just created
$ openstack server list
+--------------------------------------+------+--------+-----------------------+------------+
| ID                                   | Name | Status | Networks              | Image Name |
+--------------------------------------+------+--------+-----------------------+------------+
| ee21b545-9db0-44e0-b5c3-2c77f4f29fc6 | vm-2 | ACTIVE | internal1=192.168.7.3 | cirros     |
| 10332083-ae06-420e-846f-8a8df2fe0338 | vm-1 | ACTIVE | internal1=192.168.7.7 | cirros     |
+--------------------------------------+------+--------+-----------------------+------------+

-- check the vm counts
$# openstack hypervisor show com2.host.local
+----------------------+------------------------------------------------------------------------------------+
| Field                | Value                                                                              |
+----------------------+------------------------------------------------------------------------------------+
| aggregates           | [u'com2']                                                                          |
...snip...
| running_vms          | 2                                                                                  |
| service_host         | com2.host.local                                                                    |
...snip...
+----------------------+------------------------------------------------------------------------------------+

```

Done.


