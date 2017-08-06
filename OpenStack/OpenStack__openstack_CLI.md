## openstack CLI usage

### Introduction

The openstack CLI usage and describe the enviroments for using into OpenStack RC files.

#### Keystone credentials
Keystone public (port 5000)/admin (port 35357) end point URL should match the OS_AUTH_URL on the various enviroments.

Environment Variables | CLI options | Description
-|-|-
OS_USERNAME | --os-username | username
OS_PASSWORD | --os-password | password
OS_PROJECT_NAME | --os-project-name | project name
OS_AUTH_URL | --os-auth-url | Keystone API endpoint

