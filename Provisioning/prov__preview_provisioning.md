## Provisioning preview

### Introduction

I have managed many VMs on the various environments, and the resource (compute, storage and so on) is going to increase more and more in the future.
But we have not much engineer and time to deploy various resources, and we had to automated these tasks.
Here, I will take notes about my challenge or practice for automated provisioning of various layers.

### Provisioning layes

Layer | Descriptions | Tools
-|-|-
Provisioning OSs | Physical server or Virtual server | Cobbler, Foreman; (Kickstart, PXE, TFTP, DHCP, DNS)
Provisioning Configurations | When it boots up a OS, some meta data is configured or provisioned | Foreman + Katello, cloud-init, puppet, custom scripts
Provisioning Services | Complicated application installation over multiple servers | Terraform, Ansible (AWX), puppet, custom scripts
Provisioning Checks | Checking the provisioning results | testinfra, serverspec, custom scripts, Ansible (AWX)
Provisioning Management | Provisioning management console | Foreman + Katello, Cobbler + WEB-UI

