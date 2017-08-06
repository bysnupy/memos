## openstack CLI usage

### Introduction

The openstack CLI usage and describe the enviroments for using into OpenStack RC files.

#### openstack CLI

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
OS_PROJECT_NAME | --os-project-name | OS_TENANT_NAME, project(tenant) name)

#### Keystone
Keystone public (port 5000)/admin (port 35357) end point URL should match the OS_AUTH_URL on the various enviroments.

* Credentials

Environment Variables | CLI options | Description
-|-|-
OS_USERNAME | --os-username | username
OS_PASSWORD | --os-password | password
OS_PROJECT_NAME | --os-project-name | project name
OS_AUTH_URL | --os-auth-url | Keystone API endpoint

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


#### Service specific CLI

* nova
```
nova list
```

* neutron
```
neutron net-list
```
