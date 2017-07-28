## PoC environment construction tutorial part1

### Introduction
It is the tutorial about installing openstack for PoC environment.<br/>
The part1 is describing mainly controller and compute nodes installation.

<[The Part2 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part2.md)><br/>
<[The Part3 link](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part3.md)>

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
 
### 2. Logical nodes diagram

### 3. Installation steps

Related references: [OpenStack Docs](https://docs.openstack.org/)

#### Step0: Preparation for installation

* Installing the OpenStack RPM repository and tools on all nodes
```bash
# yum install centos-release-openstack-ocata
# yum install python-openstackclient
```
* Upgrading the system
* Ensuring a time synchronization on all nodes
* Gaining the privileged permission, such as root
* All password is same as 'poc#pass' on the all components
* Installing, configuring and enabling the MariaDB, RabbitMQ and Memcached on the controller node

```bash
# yum install mariadb mariadb-server python2-PyMySQL rabbitmq-server memcached python-memcached

-- MariaDB configuration
# cat /etc/my.cnf.d/00_openstack.cnf

[mysqld]
bind-address = 172.16.9.150
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8

-- Enabling the MariaDB service
# systemctl enable mariadb; systemctl start mariadb

-- Configuring root account password
# mysql_secure_installation

-- RabbitMQ configuration
# systemctl enable rabbitmq-server; systemctl start rabbitmq-server

-- Add the messaging server account 'openstack'
-- the hash mark which was included into the password effect the 'transport_url' parsing, so I changed the password 'poc3pass'
# rabbitmqctl add_user openstack 'poc3pass'

-- Permit configuration, write, and read access for the openstack user
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"

-- Memcached configuration
# cat /etc/sysconfig/memcached
...snip...
OPTIONS="-l 127.0.0.1,::1,172.16.9.150"
...snip...

# systemctl enable memcached; systemctl start memcached

```
* Stopping the NetworkManager service on all nodes
* Stopping the Firewalld and SELinux for proceeding smoothly
* You must be resolve the node hostnames from /etc/hosts file, as follows:

```bash
# adding at the /etc/hosts on all nodes

172.16.9.150    controller0.host.local controller0
172.16.9.151    compute1.host.local    compute1
172.16.9.152    compute2.host.local    compute2
172.16.9.153    network1.host.local    network1
172.16.9.154    block1.host.local      block1
```
Let's start out after rebooting all nodes.

#### Step1: Installing and configuring the Keystone (Identity service)

This service is installed and configured on the controller0.host.local node.

* Creating the Keystone database

```sql
MariaDB [(none)]> CREATE DATABASE keystone;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]>
```

* Installing related packages with yum

```bash
# yum install openstack-keystone httpd mod_wsgi
```

* Edit the /etc/keystone/keystone.conf as follows:

```ini
[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[cors.subdomain]
[credential]
[database]
connection = mysql+pymysql://keystone:poc#pass@controller0/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[memcache]
servers = localhost:11211
[oauth1]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
privider = fernet
[tokenless_auth]
[trust]
```

* Populate the Keystone database

```bash
# su -s /bin/sh -c "keystone-manage db_sync" keystone
# echo $?
0
#
```

* Initilize the Fernet key repositories

```bash
# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone &&
  keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
# echo $?
0
#
```

* Bootstrap the Identity service

```bash
# keystone-manage bootstrap --bootstrap-password 'poc#pass' \
  --bootstrap-admin-url http://controller0:35357/v3/ \
  --bootstrap-internal-url http://controller0:5000/v3/ \
  --bootstrap-public-url http://controller0:5000/v3/ \
  --bootstrap-region-id RegionOne
```

* Configure the httpd web server

```bash
-- set up the certain servername
# sed -i -e 's/^#ServerName.*/ServerName controller0.host.local/g' /etc/httpd/conf/httpd.conf
-- deployment wsgi virtualhost configuration as symbolic link
# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
-- enabling and starting the httpd service
# systemctl enable httpd; systemctl start httpd
```

* Define the Domain, Projects, Users and Roles

```bash
-- Gain the authorization token created earlier by exporting environment variables
$ cat adminrc
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3

$ source adminrc
-- Create the 'services' project for component services
$ openstack project create --domain default --description "Services Project" services
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Services Project                 |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 094f5e1f81164896bb437d54d3120607 |
| is_domain   | False                            |
| name        | services                         |
| parent_id   | default                          |
+-------------+----------------------------------+

-- Create the other project 'poc' for common users
$ openstack project create --domain default --description "PoC Project" poc
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | PoC Project                      |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 6a15ffaacaad4c84b78d72d740229a7b |
| is_domain   | False                            |
| name        | poc                              |
| parent_id   | default                          |
+-------------+----------------------------------+

-- Create the test user 'pocuser'
$ openstack user create --domain default --password 'poc#pass' pocuser
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | be75a0f788104233bf2bd9c6f5e37ec7 |
| name                | pocuser                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- Create the 'users' role
$ openstack role create users
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 4aff13968ee54e49bfc1ccb57b6c90aa |
| name      | users                            |
+-----------+----------------------------------+

-- Add the users role to pocuser
$ openstack role add --project poc --user pocuser users
```

* Verifying the Keystone installation

Edit the /etc/keystone/keystone-paste.ini as follows (for disabling the temporary authentication token mechanism):
remove admin_token_auth from the [pipeline:public_api], [pipeline:admin_api], and [pipeline:api_v3] sections

```bash
# sed -i -e 's/ admin_token_auth / /g' /etc/keystone/keystone-paste.ini
```

As the users created earlier, issue the authentication tokens

```bash
$ source adminrc
$ unset OS_AUTH_URL OS_PASSWORD
$ openstack --os-auth-url http://controller0:35357/v3   --os-project-domain-name default \
            --os-user-domain-name default   --os-project-name admin --os-username admin \
            --os-password 'poc#pass'  token issue 
+------------+-------------------------------------------------------------+
| Field      | Value                                                       |
+------------+-------------------------------------------------------------+
| expires    | 2017-07-19T09:39:40+0000                                    |
| id         | gAAAAABZbxrMKu_ktk1J8GkT61eZwfDLykdWtTlA7DaWifIQYMePFUxfKai |
|            | KYV585Y4lpCQBhDBV9jxxWnLuwJFSZmhE1CcnBXQBcGs_q71GsdmPGgn-k5 |
|            | d8txm_1HRCHydHM7zyvSRRlCzhT8euDjRCdkTtqcGcyDIkTnuy5XLh0haUP |
|            | GLFn8Y                                                      |
| project_id | 6f173e1f310146c9883a2c50d1336bc0                            |
| user_id    | ad1c749cd6474ae3a5b61f0ea3f05c22                            |
+------------+-------------------------------------------------------------+

$ openstack --os-auth-url http://controller0:5000/v3   --os-project-domain-name default \
            --os-user-domain-name default   --os-project-name poc --os-username pocuser \
            --os-password 'poc#pass'  token issue
+------------+-------------------------------------------------------------+
| Field      | Value                                                       |
+------------+-------------------------------------------------------------+
| expires    | 2017-07-19T09:40:52+0000                                    |
| id         | gAAAAABZbxsUG0Sw9ANdJ66l8NkzdyRcRkWxezU5XtB7jOLh6MEJ1Q51U5L |
|            | hrNoC6_Hr66H3r6d4ASwX7yXU1-O_C4PFND0QdhFpZt_nvSWVUBSZHJpKLt |
|            | 1Ocwd4BUTVBnNxzyKb5ENfx2fotLDxBcMh4cdG9yNOIJoLXmkOMIKr-     |
|            | 4IjPb0v-7c                                                  |
| project_id | 6a15ffaacaad4c84b78d72d740229a7b                            |
| user_id    | be75a0f788104233bf2bd9c6f5e37ec7                            |
+------------+-------------------------------------------------------------+
```

#### Step2: Creating the OpenRC for using openstack client command simply

* admin user environment script
```bash
$ cat > ~/admin-openrc <<EOF
# admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD='poc#pass'
export OS_AUTH_URL=http://controller0:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

* pocuser user environment script
```bash
$ cat > ~/pocuser-openrc <<EOF
# pocuser-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=poc
export OS_USERNAME=pocuser
export OS_PASSWORD='poc#pass'
export OS_AUTH_URL=http://controller0:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```

#### Step3: Installing and configuring the Glance (Image service)

This service is also installed and configured on the controller0.host.local node.

* Creating the Glance database

```sql
MariaDB [(none)]> CREATE DATABASE glance;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]>
```

* Configuring the credentials for service

```bash
$ source ~/admin-openrc
$ openstack user create --domain default --password 'poc#pass' glance
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 76a296fa82db438d9061e77556e2beaf |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- role setting
$ openstack role add --project services --user glance admin

-- creating image service
$ openstack service create --name glance --description "OpenStack Image" image
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | 05c57a54d90f47f4bf8b650e5ab69148 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+

-- setting for public, internal and admin endpoint entities
$ openstack endpoint create --region RegionOne image public http://controller0:9292
$ openstack endpoint create --region RegionOne image internal http://controller0:9292
$ openstack endpoint create --region RegionOne image admin http://controller0:9292
```

* Installing related packages with yum

```bash
# yum install openstack-glance
```

* Edit the /etc/glance/glance-api.conf as follows:

```ini
[DEFAULT]
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://glance:poc#pass@controller0/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = glance
password = poc#pass
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
```
* Edit the /etc/glance/glance-registry.conf as follows:

```ini
[DEFAULT]
[database]
connection = mysql+pymysql://glance:poc#pass@controller0/glance
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = glance
password = poc#pass
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
```

* Populate the Glance database
```bash
# su -s /bin/sh -c "glance-manage db_sync" glance
```

* Starting and enabling the Glance services

```bash
# systemctl enable openstack-glance-api openstack-glance-registry
...snip...
# systemctl start openstack-glance-api openstack-glance-registry
```

* Verifying the Glance installation

```bash
$ source ~/admin-openrc
$ curl -O http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
...snip...
$ openstack image create "cirros" --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare --public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                     |
| container_format | bare                                                 |
| created_at       | 2017-07-19T09:50:28Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/3dd7948f-6c14-4aaf-802d-be2716a1ad4a/file |
| id               | 3dd7948f-6c14-4aaf-802d-be2716a1ad4a                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros                                               |
| owner            | 6f173e1f310146c9883a2c50d1336bc0                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13267968                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2017-07-19T09:50:29Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```

#### Step4: Installing and configuring the Nova (Compute service) - Controller node tasks

This service is installed and configured on the controller0.host.local, compute1.host.local and compute2.host.local nodes.

This section describes the tasks on the controller0.host.local node.

* Creating databases for compute service

```sql
MariaDB [(none)]> CREATE DATABASE nova_api;
MariaDB [(none)]> CREATE DATABASE nova;
MariaDB [(none)]> CREATE DATABASE nova_cell0;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'poc#pass';
MariaDB [(none)]> FLUSH PRIVILEGES;
```

* Define the essential objects for compute service

```bash
$ source ~/admin-openrc
-- configuration about user
$ openstack user create --domain default --password 'poc#pass' nova
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | b3cc846f0a26467c88a8d00f675656de |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

$ openstack role add --project services --user nova admin

-- Create the compute service
$ openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 312b8fd9b3a04dca9bfc95cca3cad65c |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
-- create the each endpoint (skipped the result output)
$ openstack endpoint create --region RegionOne compute public http://controller0:8774/v2.1
$ openstack endpoint create --region RegionOne compute internal http://controller0:8774/v2.1
$ openstack endpoint create --region RegionOne compute admin http://controller0:8774/v2.1
```

* Cofiguration for placement service

```bash
-- Create the credential
$ openstack user create --domain default --password 'poc#pass' placement
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 06e0c5e36b7947aa9c6f4a7fc5b25ee2 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

-- add the placement user to admin role
$ openstack role add --project services --user placement admin

-- Create the placement API service
$ openstack service create --name placement --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | bd9661a98a51483fb5e503d698ed2f88 |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+

-- create each endporint of the service (skipped the result output)
$ openstack endpoint create --region RegionOne placement public http://controller0:8778
$ openstack endpoint create --region RegionOne placement internal http://controller0:8778
$ openstack endpoint create --region RegionOne placement admin http://controller0:8778
```

* Installing and configuring related components

```bash
# yum install openstack-nova-api openstack-nova-conductor openstack-nova-console \
              openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api
```

* Edit the /etc/nova/nova.conf file as follows:

```ini
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:poc3pass@controller0
my_ip = 172.16.9.150
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:poc#pass@controller0/nova_api
[barbican]
[cache]
[cells]
[cinder]
[cloudpipe]
[conductor]
[console]
[consoleauth]
[cors]
[cors.subdomain]
[crypto]
[database]
connection = mysql+pymysql://nova:poc#pass@controller0/nova
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://controller0:9292
[guestfs]
[healthcheck]
[hyperv]
[image_file_url]
[ironic]
[key_manager]
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = nova
password = poc#pass
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = services
auth_type = password
user_domain_name = Default
auth_url = http://controller0:35357/v3
username = placement
password = poc#pass
[quota]
[rdp]
[remote_debug]
[scheduler]
[serial_console]
[service_user]
[spice]
[ssl]
[trusted_computing]
[upgrade_levels]
[vendordata_dynamic_auth]
[vmware]
[vnc]
enabled = true
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
[workarounds]
[wsgi]
[xenserver]
[xvp]
```

* Adding the configuration to /etc/httpd/conf.d/00-nova-placement-api.conf file and restart the service

```httpd
...snip...
<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>

$ systemctl restart httpd
```

* Populate the nova_api database, check the result from /var/log/nova/nova-manage.log

```bash
# su -s /bin/sh -c "nova-manage api_db sync" nova
```

* Register the cell0 database

```bash
# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```

* Create the cell1 cell
```bash
# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
bfd3180c-6902-4ab1-89db-9aec3d45d631

-- verifying the registered status
# nova-manage cell_v2 list_cells
+-------+--------------------------------------+
|  Name |                 UUID                 |
+-------+--------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |
| cell1 | bfd3180c-6902-4ab1-89db-9aec3d45d631 |
+-------+--------------------------------------+
```

* Populate the nova database

:warning:If you are encountered "ERROR: could not access cell mapping database - has api db been created?" message,
you can check the 'nova_api.cell_mapping' table and check 'database_connection' column.

```bash
# su -s /bin/sh -c "nova-manage db sync" nova
```

* Enabling and starting the related component services

```bash
# systemctl enable openstack-nova-api openstack-nova-consoleauth \
                   openstack-nova-scheduler openstack-nova-conductor openstack-nova-novncproxy
# systemctl start openstack-nova-api openstack-nova-consoleauth \
                  openstack-nova-scheduler openstack-nova-conductor openstack-nova-novncproxy                   
```

#### Step5: Installing and configuring the Nova (Compute service) - Compute nodes tasks

This section describes the compute nodes installation on compute1.host.local and compute2.host.local nodes.

* Packages installation

```bash
# yum install openstack-nova-compute
```

* Edit the /etc/nova/nova.conf files on the both nodes

```ini
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:poc3pass@controller0
# compute1 
my_ip = 172.16.9.151
# if the node is compute2, you'll remove the comment the bellow.
#my_ip = 172.16.9.152
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
[barbican]
[cache]
[cells]
[cinder]
[cloudpipe]
[conductor]
[console]
[consoleauth]
[cors]
[cors.subdomain]
[crypto]
[database]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://controller0:9292
[guestfs]
[healthcheck]
[hyperv]
[image_file_url]
[ironic]
[key_manager]
[keystone_authtoken]
auth_uri = http://controller0:5000
auth_url = http://controller0:35357
memcached_servers = controller0:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = services
username = nova
password = poc#pass
[libvirt]
# if vmx or smx flag is included into /proc/cpuinfo, you need not a following line.
virt_type = qemu
[matchmaker_redis]
[metrics]
[mks]
[neutron]
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = services
auth_type = password
user_domain_name = Default
auth_url = http://controller0:35357/v3
username = placement
password = poc#pass
[quota]
[rdp]
[remote_debug]
[scheduler]
[serial_console]
[service_user]
[spice]
[ssl]
[trusted_computing]
[upgrade_levels]
[vendordata_dynamic_auth]
[vmware]
[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller0:6080/vnc_auto.html
keymap = ja
[workarounds]
[wsgi]
[xenserver]
[xvp]
```

* Enabling and starting the related services

```bash
# systemctl enable libvirtd openstack-nova-compute
# systemctl start libvirtd openstack-nova-compute
```

#### Step5: Installing and configuring the Nova (Compute service) - Verifying the installation on the Controllor node

Some remaining configuration and verifying tasks are conducted on the controller0.host.local node.

* Confirm the running compute nodes
```bash
$ source ~/admin-openrc
$ openstack hypervisor list
+----+---------------------+-----------------+--------------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP      | State |
+----+---------------------+-----------------+--------------+-------+
|  1 | compute2.host.local | QEMU            | 172.16.9.152 | up    |
|  2 | compute1.host.local | QEMU            | 172.16.9.151 | up    |
+----+---------------------+-----------------+--------------+-------+
```

* Adding the compute nodes to cell database

```bash
# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting compute nodes from cell 'cell1': bfd3180c-6902-4ab1-89db-9aec3d45d631
Found 2 computes in cell: bfd3180c-6902-4ab1-89db-9aec3d45d631
Checking host mapping for compute host 'compute2.host.local': 3fdabcb0-dd69-4481-9cb2-5c640eec1463
Creating host mapping for compute host 'compute2.host.local': 3fdabcb0-dd69-4481-9cb2-5c640eec1463
Checking host mapping for compute host 'compute1.host.local': 6b274561-9206-4132-812b-1dd012004ee3
Creating host mapping for compute host 'compute1.host.local': 6b274561-9206-4132-812b-1dd012004ee3
```

* Verifying the compute node installation

```bash
$ source ~/admin-openrc

-- list the compute service components
$ openstack compute service list
+----+------------------+------------------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host                   | Zone     | Status  | State | Updated At                 |
+----+------------------+------------------------+----------+---------+-------+----------------------------+
|  1 | nova-conductor   | controller0.host.local | internal | enabled | up    | 2017-07-20T08:49:21.000000 |
|  2 | nova-scheduler   | controller0.host.local | internal | enabled | up    | 2017-07-20T08:49:21.000000 |
|  3 | nova-consoleauth | controller0.host.local | internal | enabled | up    | 2017-07-20T08:49:22.000000 |
|  6 | nova-compute     | compute2.host.local    | nova     | enabled | up    | 2017-07-20T08:49:22.000000 |
|  7 | nova-compute     | compute1.host.local    | nova     | enabled | up    | 2017-07-20T08:49:23.000000 |
+----+------------------+------------------------+----------+---------+-------+----------------------------+

-- check the placement API and cell status (as root user)
# nova-status upgrade check
+---------------------------+
| Upgrade Check Results     |
+---------------------------+
| Check: Cells v2           |
| Result: Success           |
| Details: None             |
+---------------------------+
| Check: Placement API      |
| Result: Success           |
| Details: None             |
+---------------------------+
| Check: Resource Providers |
| Result: Success           |
| Details: None             |
+---------------------------+
```

To continue to <[Part2](https://github.com/bysnupy/memos/blob/master/OpenStack/OpenStack__ocata_poc_part2.md)>
