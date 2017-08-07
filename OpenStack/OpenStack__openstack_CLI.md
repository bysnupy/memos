## openstack CLI usage

### Introduction

The openstack CLI usage and description for the enviroment variables for using into OpenStack RC files.

#### openstack CLI

Package name: python-openstackclient

```
openstack OPTIONS COMMAND ACTION ARGUMENTS
```

* Help
```
openstack help
openstack help COMMAND
oepnstack help COMMAND ACTION
```

* Command

Command | Description
-|-
user | user management
project | project management
server | instance management
network | network management
image | image management
flavor | flavor management
volume | volume management
floating ip | floating ip management

* Action

Action | Description
-|-
create | the target resource create
delete | the target resource delete
list | list the resources
show | show the target resource details
set | update the target resource values

#### Tenant information

Environment Variables | CLI options | Descrtipion
-|-|-
OS_PROJECT_NAME | --os-project-name | OS_TENANT_NAME, project(tenant) name)
OS_REGION_NAME | --os-region-name | region name

#### Keystone

Keystone public (port 5000)/admin (port 35357) end point URL should match the OS_AUTH_URL on the various enviroments.

* Credentials

Environment Variables | CLI options | Description
-|-|-
OS_USERNAME | --os-username | username
OS_PASSWORD | --os-password | password
OS_PROJECT_NAME | --os-project-name | project name
OS_AUTH_URL | --os-auth-url | Keystone API endpoint

:start:Keystone API versions, v2.0 and v3.0 are available. v3.0 is based on Project Domain.

Version|OS_AUTH_URL|OS_IDENTITY_API_VERSION|Description
-|-|-|-
v2.0|http://keystone-endpoint:5000/v2.0 | 2 | OS_IDENTITY_API_VERSION is optional
v3.0|http://keystone-endpoint:5000/v3.0 | 3 | OS_IDENTITY_API_VERSION is required, if you want to use the version 3.0

* Token

OpenStack use the unique access code for security to request the tasks to end point API.
The access code is valid for a limited period time, it has the role (authorization) information either.

```
openstack token issue
```

#### Flavor

* List the flavors

```
openstack falvor list
```

* Show the details

```
openstack flavor show FLAVORNAME
```

#### Project

* Show the details

```
openstack project show PROJECTNAME
```

#### User

* Show the details

```
openstack user show USERNAME
```

#### Image

* list the images

```
openstack image list
```

#### Server (Instance)

* list the instances

```
openstack server list
```

* Show the instance details

```
openstack server show INSTANCENAME
```

* Set server properties

```
-- root password changing inside an instance 
openstack server set --root-password INSTANCENAME
```

* Launching new instance

```
openstack server create --flavor FLAVORNAME --image IMAGENAME --nic net-id=NETWORKNAME/ID INSTANCENAME 
```

* Stop the instance

```
openstack server stop INSTANCENAME/ID
```

* Remove the instance

```
openstack server delete INSTANCENAME/ID
```

* Show the console log

```
openstack console log show INSTANCENAME/ID
```

* Display VNC console URL

```
openstack console url show INSTANCENAME/ID
```

#### Service specific CLI

* nova

```
nova list
```

* neutron

```
neutron net-list
```
