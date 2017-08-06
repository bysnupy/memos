## The comparison of launching instances

#### Traditional virtualization platform

The following steps take time.

* Define the virtual server specifications (virtual hardware specs, such as CPU, Memory and so on)
* Define the storage for VMs
* Installing OS

#### OpenStack and Cloud platform

Using virtual machine image templates, so not just one instance but also more instances can use same images.

* The instance start up with copying an existing image(installed OS in advance)
* base image are not affected from changes
* fisrt time take time to launch instance for copying image but second time does not.
