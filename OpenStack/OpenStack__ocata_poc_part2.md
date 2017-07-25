## PoC environment construction tutorial part2

### Introduction
It is the tutorial about installing openstack for PoC environment.<br/>
The part2 is describing mainly network and dashboard services installation.

<[The Part1 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part1.md)>

### 1. Environment information
:star: The nodes are virtual machines based on KVM.
* OpenStack version: Ocata
* OS: CentOS 7.3
* Deployment method: RDO
* Server Specifications (total: 5 nodes):

Nodes|CPU|Memory|NICs|Storages
-|-|-|-|-
controller x1 | Core x4| 4 GiB| NIC x1| 50 GB
compute x2    | Core x4|4 GiB| NIC x3|50 GB
network x1    |Core x2|4 GiB |NIC x3 | 30 GB
block x1      |Core x2|4 GiB |NIC x2 | 30 GB (+secondary volume 30 GB)

* IPs and interfaces table

Hostname|Components|IPs|Interfaces
--------|----|--|---------
controller0.host.local|Horizon<br/>Keystone<br/>Glance<br/>Management APIs<br/>MariaDB<br/>RabbitMQ<br/>Memcached|eth0: 172.16.9.150|eth0
compute1.host.local|Nova|eth0: 172.16.9.151<br/>eth1: 192.168.9.151|eth0<br/>eth1<br/>eth2<br/>
compute2.host.local|Nova|eth0: 172.16.9.152<br/>eth1: 192.168.9.152|eth0<br/>eth1<br/>eth2<br/>
network1.host.local|Neutron|eth0: 172.16.9.153<br/>eth1: 192.168.9.153|eth0<br/>eth1<br/>eth2<br/>
block1.host.local|Cinder|eth0: 172.16.9.154<br/>eth1: 192.168.9.154|eth0<br/>eth1

#### Step0: Prerequisites the following installation

You must be done all tasks in the [part1](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part1.md).

#### Step1: Installing and configuring the Neutron (Networking service) on Controller node
The following tasks should be done on the controller0 node.

* Creating the database for Neutron service

```sql
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'poc#pass';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'poc#pass';
FLUSH PRIVILEGES;
```

* Defining the service and credentials

```bash
$ source ~/admin-openrc
-- Create the user
$ openstack user create --domain default --password 'poc#pass' neutron
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | dc6bbb73167c488497dc7b9aa9e87a01 |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- Adding the user to admin role
$ openstack role add --project services --user neutron admin

-- Create the service
$ openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | b7551a74442546fe848f776bd9bf6678 |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+

-- Register each API endpoints (skipped the result output)
$ openstack endpoint create --region RegionOne network public http://controller0:9696
$ openstack endpoint create --region RegionOne network internal http://controller0:9696
$ openstack endpoint create --region RegionOne network admin http://controller0:9696
```

* Installing the packages through yum on the controller0.host.local node

```bash
# yum install openstack-neutron openstack-neutron-ml2
```

* Edit /etc/neutron/neutron.conf file as follows

```ini
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:poc3pass@controller0
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[agent]
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://neutron:poc#pass@controller0/neutron
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = neutron
password = poc#pass
[matchmaker_redis]
[nova]
auth_url = http://controller0:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = services
username = nova
password = poc#pass
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]
```

* Edit the /etc/neutron/plugins/ml2/ml2_conf.ini

```ini
[DEFAULT]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
flat_networks = external_network
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = true
```

* Create the symbolic link to /etc/neutron/plugin.ini

```bash
# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

* Edit the /etc/nova/nova.conf file on controller0 node and restart the nova-api service

```ini
..snip...
[neutron]
url = http://controller0:9696
auth_url = http://controller0:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = services
username = neutron
password = poc#pass
service_metadata_proxy = True
metadata_proxy_shared_secret = poc#pass
...snip...
```

The new settings take effect after restarting

```bash
# systemctl restart openstack-nova-api
```

* Service enabling and starting

```bash
# systemctl enable neutron-server
# systemctl start neutron-server
```

#### Step2: Installing and configuring the Neutron (Networking service) on Networking node
The following tasks should be done on the network1 node.

* Installing the packages through yum command

```bash
# yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

* Edit /etc/neutron/neutron.conf same as controller0 node

```ini
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:poc3pass@controller0
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[agent]
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://neutron:poc#pass@controller0/neutron
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = neutron
password = poc#pass
[matchmaker_redis]
[nova]
auth_url = http://controller0:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = services
username = nova
password = poc#pass
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]
```

* Edit the /etc/neutron/plugins/ml2/ml2_conf.ini same as controller0 node

```ini
[DEFAULT]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
flat_networks = external_network
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = true
```

* Edit the /etc/neutron/plugins/ml2/linuxbridge_agent.ini file as follows

```ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = external_network:eth2
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = True
local_ip = 192.168.9.153
l2_population = True
```

* Edit the /etc/neutron/l3_agent.ini as follows

```ini
[DEFAULT]
interface_driver = linuxbridge
[agent]
[ovs]
```

* Edit the /etc/neutron/dhcp_agent.ini file

```ini
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
[agent]
[ovs]
```

* Edit the /etc/neutron/metadata_agent.ini file

```ini
[DEFAULT]
nova_metadata_ip = controller0
metadata_proxy_shared_secret = poc#pass
[agent]
[cache]
```

* Create the symbolic link to /etc/neutron/plugin.ini

```bash
# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

* Initilize the neutron database

```bash
# su -s /bin/sh -c \
  "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" \
   neutron
```

* Enabling and starting the services

```bash
# systemctl enable neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent neutron-l3-agent
# systemctl start neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent neutron-l3-agent
```

#### Step3: Installing and configuring the Neutron (Networking service) on Compute nodes
The following tasks should be done on the compute1 and compute2 nodes.

* Installing the needed packages

```bash
# yum install openstack-neutron-linuxbridge ebtables ipset
```

* Edit the /etc/neutron/neutron.conf file as follows on the both compute1 and compute2 nodes 

```ini
[DEFAULT]
transport_url = rabbit://openstack:poc3pass@controller0
auth_strategy = keystone
[agent]
[cors]
[cors.subdomain]
[database]
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = neutron
password = poc#pass
[matchmaker_redis]
[nova]
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]
```

* Edit the /etc/neutron/plugins/ml2/linuxbridge_agent.ini file as follows

```ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = external_network:eth2
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = true
# in case of compute2 node, local_ip = 192.168.9.152 
local_ip = 192.168.9.151
l2_population = true
```

* Edit the neutron section of /etc/nova/nova.conf file and restart the nova-compute service

```ini
...snip...
[neutron]
url = http://controller0:9696
auth_url = http://controller0:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = services
username = neutron
password = poc#pass
...snip...
```

restarting the nova-compute service on the compute1 and compute2 nodes

```bash
# systemctl restart openstack-nova-compute
```

* Enabling and starting the linux bridge agent service on both nodes

```
# systemctl enable neutron-linuxbridge-agent
# systemctl start neutron-linuxbridge-agent
```

#### Step4: Verifying the networking service installation
Perform the following commands on the controller0 node.

* Listing the extensions from neutron-server service

```bash
$ source ~/admin-openrc
$ openstack extension list --network
+---------------------------+---------------------------+----------------------------+
| Name                      | Alias                     | Description                |
+---------------------------+---------------------------+----------------------------+
| Default Subnetpools       | default-subnetpools       | Provides ability to mark   |
|                           |                           | and use a subnetpool as    |
|                           |                           | the default                |
| Network IP Availability   | network-ip-availability   | Provides IP availability   |
|                           |                           | data for each network and  |
|                           |                           | subnet.                    |
| Network Availability Zone | network_availability_zone | Availability zone support  |
|                           |                           | for network.               |
| Auto Allocated Topology   | auto-allocated-topology   | Auto Allocated Topology    |
| Services                  |                           | Services.                  |
...snip...

-- check the agents status
$ openstack network agent list
+-----------------------+--------------------+---------------------+-------------------+-------+-------+--------------------------+
| ID                    | Agent Type         | Host                | Availability Zone | Alive | State | Binary                   |
+-----------------------+--------------------+---------------------+-------------------+-------+-------+--------------------------+
| 1160160c-4d70-401a-   | L3 agent           | network1.host.local | nova              | True  | UP    | neutron-l3-agent         |
| 99fc-ab441ee006cc     |                    |                     |                   |       |       |                          |
| 52a28e46-22f7-4853-87 | Linux bridge agent | compute1.host.local | None              | True  | UP    | neutron-linuxbridge-     |
| 41-1eda0bbdb873       |                    |                     |                   |       |       | agent                    |
| 5dbba274-096f-44f2-a0 | Metadata agent     | network1.host.local | None              | True  | UP    | neutron-metadata-agent   |
| 69-3c8abc92ff9c       |                    |                     |                   |       |       |                          |
| 69284630-f58c-4ffa-a6 | Linux bridge agent | compute2.host.local | None              | True  | UP    | neutron-linuxbridge-     |
| 78-e305cedcd3db       |                    |                     |                   |       |       | agent                    |
| 9b33c8c3-9bf5-4f49-be | Linux bridge agent | network1.host.local | None              | True  | UP    | neutron-linuxbridge-     |
| 38-ae22c8424d99       |                    |                     |                   |       |       | agent                    |
| e24662fd-9d3b-4be4-83 | DHCP agent         | network1.host.local | nova              | True  | UP    | neutron-dhcp-agent       |
| b6-6ce862de52f5       |                    |                     |                   |       |       |                          |
+-----------------------+--------------------+---------------------+-------------------+-------+-------+--------------------------+

```

#### Step5: Installing and configuring the Horizon (Dashboard service) on Controller node
The following tasks should be done on the controller0 node.

* Installing the packages

```bash
# yum install openstack-dashboard
```

* Edit the /etc/openstack-dashboard/local_settings file

```python
import os
from django.utils.translation import ugettext_lazy as _
from openstack_dashboard.settings import HORIZON_CONFIG
DEBUG = False
WEBROOT = '/dashboard/'
ALLOWED_HOSTS = ['*', 'localhost']
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
LOCAL_PATH = '/tmp'
SECRET_KEY='0129b1a80de9ded5d33a'
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller0:11211',
    }
}
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
OPENSTACK_HOST = "controller0"
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "users"
OPENSTACK_KEYSTONE_BACKEND = {
    'name': 'native',
    'can_edit_user': True,
    'can_edit_group': True,
    'can_edit_project': True,
    'can_edit_domain': True,
    'can_edit_role': True,
}
OPENSTACK_HYPERVISOR_FEATURES = {
    'can_set_mount_point': False,
    'can_set_password': False,
    'requires_keypair': False,
    'enable_quotas': True
}
OPENSTACK_CINDER_FEATURES = {
    'enable_backup': False,
}
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': True,
    'enable_quotas': True,
    'enable_ipv6': True,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': True,
    'enable_firewall': True,
    'enable_vpn': True,
    'enable_fip_topology_check': True,
    'profile_support': None,
    'supported_vnic_types': ['*'],
}
OPENSTACK_HEAT_STACK = {
    'enable_user_pass': True,
}
IMAGE_CUSTOM_PROPERTY_TITLES = {
    "architecture": _("Architecture"),
    "kernel_id": _("Kernel ID"),
    "ramdisk_id": _("Ramdisk ID"),
    "image_state": _("Euca2ools state"),
    "project_id": _("Project ID"),
    "image_type": _("Image Type"),
}
IMAGE_RESERVED_CUSTOM_PROPERTIES = []
API_RESULT_LIMIT = 1000
API_RESULT_PAGE_SIZE = 20
SWIFT_FILE_TRANSFER_CHUNK_SIZE = 512 * 1024
INSTANCE_LOG_LENGTH = 35
DROPDOWN_MAX_ITEMS = 30
TIME_ZONE = "Asia/Tokyo"
POLICY_FILES_PATH = '/etc/openstack-dashboard'
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'operation': {
            'format': '%(asctime)s %(message)s'
        },
    },
    'handlers': {
        'null': {
            'level': 'DEBUG',
            'class': 'logging.NullHandler',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
        },
        'operation': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'operation',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['null'],
            'propagate': False,
        },
        'requests': {
            'handlers': ['null'],
            'propagate': False,
        },
        'horizon': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'horizon.operation_log': {
            'handlers': ['operation'],
            'level': 'INFO',
            'propagate': False,
        },
        'openstack_dashboard': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'novaclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'cinderclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'keystoneclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'glanceclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'neutronclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'heatclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'swiftclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'openstack_auth': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'nose.plugins.manager': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'django': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'iso8601': {
            'handlers': ['null'],
            'propagate': False,
        },
        'scss': {
            'handlers': ['null'],
            'propagate': False,
        },
    },
}
SECURITY_GROUP_RULES = {
    'all_tcp': {
        'name': _('All TCP'),
        'ip_protocol': 'tcp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_udp': {
        'name': _('All UDP'),
        'ip_protocol': 'udp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_icmp': {
        'name': _('All ICMP'),
        'ip_protocol': 'icmp',
        'from_port': '-1',
        'to_port': '-1',
    },
    'ssh': {
        'name': 'SSH',
        'ip_protocol': 'tcp',
        'from_port': '22',
        'to_port': '22',
    },
    'smtp': {
        'name': 'SMTP',
        'ip_protocol': 'tcp',
        'from_port': '25',
        'to_port': '25',
    },
    'dns': {
        'name': 'DNS',
        'ip_protocol': 'tcp',
        'from_port': '53',
        'to_port': '53',
    },
    'http': {
        'name': 'HTTP',
        'ip_protocol': 'tcp',
        'from_port': '80',
        'to_port': '80',
    },
    'pop3': {
        'name': 'POP3',
        'ip_protocol': 'tcp',
        'from_port': '110',
        'to_port': '110',
    },
    'imap': {
        'name': 'IMAP',
        'ip_protocol': 'tcp',
        'from_port': '143',
        'to_port': '143',
    },
    'ldap': {
        'name': 'LDAP',
        'ip_protocol': 'tcp',
        'from_port': '389',
        'to_port': '389',
    },
    'https': {
        'name': 'HTTPS',
        'ip_protocol': 'tcp',
        'from_port': '443',
        'to_port': '443',
    },
    'smtps': {
        'name': 'SMTPS',
        'ip_protocol': 'tcp',
        'from_port': '465',
        'to_port': '465',
    },
    'imaps': {
        'name': 'IMAPS',
        'ip_protocol': 'tcp',
        'from_port': '993',
        'to_port': '993',
    },
    'pop3s': {
        'name': 'POP3S',
        'ip_protocol': 'tcp',
        'from_port': '995',
        'to_port': '995',
    },
    'ms_sql': {
        'name': 'MS SQL',
        'ip_protocol': 'tcp',
        'from_port': '1433',
        'to_port': '1433',
    },
    'mysql': {
        'name': 'MYSQL',
        'ip_protocol': 'tcp',
        'from_port': '3306',
        'to_port': '3306',
    },
    'rdp': {
        'name': 'RDP',
        'ip_protocol': 'tcp',
        'from_port': '3389',
        'to_port': '3389',
    },
}
REST_API_REQUIRED_SETTINGS = ['OPENSTACK_HYPERVISOR_FEATURES',
                              'LAUNCH_INSTANCE_DEFAULTS',
                              'OPENSTACK_IMAGE_FORMATS',
                              'OPENSTACK_KEYSTONE_DEFAULT_DOMAIN']
ALLOWED_PRIVATE_SUBNET_CIDR = {'ipv4': [], 'ipv6': []}
```

* Restart the related services

```
# systemctl restart httpd memcached
```

* Verifying the Hozion installation

Accessing the URL http://controller0.host.local/dashboard/ through browser

Login page
![Horizon login1](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_horizon1.png)

After login

![Horizon login2](https://github.com/bysnupy/memos/blob/master/OpenStack/images/OpenStack__ocata_poc_horizon2.png)


To continue to <[Part3](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part3.md)>
