## Building the provisioning and system life cycle management system with Foreman and Katello components

### Introduction

If you are boring for provisioning tasks with Cobbler, 
I recommend that creating new provisioning system with new components, such as Foreman and Katello.
These components can do more than provisioning tasks. 

### Environment

* Components

Component | Description
-|-
Foreman | System life cycle management tool, configuring and monitoring of physical and virtual servers.
Katello | Content management tool

* Components features

Component | Features
-|-
Foreman | Provisioning (bare-metal, virtualization, cloud) 
Katello | Local repository management (yum, puppet), Snapshot (content, configuration)

* Node and version infomation

Field | Value
-|-
Hostname | prov.host.local
CPU | 4 Cores
MEM | 4 GB
OS  | CentOS 7.4
Foreman | 1.15
Katello | 3.4

### Installation steps

Reference: [ Foreman and Katello official documentation ](https://theforeman.org/documentation.html)

#### Step1: Installation Foreman and Katello with yum and puppet

* Configuration of yum repositories

```bash
--- deploy puppet labs repository for latest puppet installation
yum -y install https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm

--- enable EPEL repository
yum -y install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

--- enable Foreman repository
yum -y install https://yum.theforeman.org/releases/1.15/el7/x86_64/foreman-release.rpm

--- enable Katello repository
yum -y localinstall http://fedorapeople.org/groups/katello/releases/yum/3.4/katello/el7/x86_64/katello-repos-latest.rpm

--- deploy Foreman software collection repository
yum -y install foreman-release-scl

--- installation of foreman installer
yum -y install foreman-installer

--- update repositories
yum -y update

```
* Installation of Katello through yum

```
yum -y install katello
```

* Installing with foreman-installer tool

```

```
