## PoC environment construction tutorial

### Introduction
It is the tutorial about installing openstack for PoC environment.

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
# rabbitmqctl add_user openstack 'poc#pass'

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
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'poc#pass';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'poc#pass';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

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
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'poc#pass';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glacne'@'%' IDENTIFIED BY 'poc#pass';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

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
...snip...
$ openstack endpoint create --region RegionOne image internal http://controller0:9292
...snip...
$ openstack endpoint create --region RegionOne image admin http://controller0:9292
...snip...
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

#### Step4: Installing and configuring the Nova (Compute service)

This service is installed and configured on the controller0.host.local, compute1.host.local and compute2.host.local nodes.

This section describes the tasks on the controller0.host.local node.

