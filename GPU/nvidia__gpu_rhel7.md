# GPU install steps on RHEL7

## Install Related links:
https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#introduction
https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=RHEL&target_version=7&target_type=rpmnetwork
```cmd
sudo yum-config-manager --add-repo http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-rhel7.repo
sudo yum clean all
sudo yum -y install nvidia-driver-latest-dkms cuda
sudo yum -y install cuda-drivers
```

## Installation steps
```cmd
# subscription-manager repos --enable rhel-7-server-e4s-optional-rpms

# yum -y install kernel-devel-`uname -r`
# yum install -y \
https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# yum install -y \
https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-repo-rhel7-10.0.130-1.x86_64.rpm
# yum -y install nvidia-driver nvidia-driver-cuda nvidia-modprobe

#  modprobe -r nouveau
# lsmod | grep nvidia
nvidia_drm             43690  0 
nvidia_modeset       1109452  1 nvidia_drm
nvidia              19893032  1 nvidia_modeset
ipmi_msghandler        56728  2 ipmi_devintf,nvidia
drm_kms_helper        186531  2 cirrus,nvidia_drm
drm                   456166  5 ttm,drm_kms_helper,cirrus,nvidia_drm

# nvidia-modprobe && nvidia-modprobe -u
# echo $?
0

# lsmod | grep nvidia
nvidia_uvm            930831  0 
nvidia_drm             43690  0 
nvidia_modeset       1109452  1 nvidia_drm
nvidia              19893032  2 nvidia_modeset,nvidia_uvm
ipmi_msghandler        56728  2 ipmi_devintf,nvidia
drm_kms_helper        186531  2 cirrus,nvidia_drm
drm                   456166  5 ttm,drm_kms_helper,cirrus,nvidia_drm

# nvidia-smi --query-gpu=gpu_name --format=csv,noheader --id=0 | sed -e 's/ /-/g'
Tesla-V100-SXM2-16GB

# curl -s -L \
https://nvidia.github.io/nvidia-container-runtime/centos7/nvidia-container-runtime.repo | tee /etc/yum.repos.d/nvidia-container-runtime.repo

# yum -y install nvidia-container-runtime-hook

# cat <<EOF >> /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json
{
"hasbindmounts": true,
"hook": "/usr/bin/nvidia-container-runtime-hook",
"stage": [ "prestart" ]
}
EOF

# wget \
https://raw.githubusercontent.com/zvonkok/origin-ci-gpu/master/selinux/nvidia-container.pp

# semodule -i nvidia-container.pp

# nvidia-container-cli -k list | restorecon -v -f -

# restorecon -Rv /dev

# restorecon -Rv /var/lib/kubelet
restorecon:  lstat(/var/lib/kubelet) failed:  No such file or directory

# yum install nvidia-driver-latest-dkms
# yum install cuda
# yum -y install cuda-drivers

# cat /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json
{
   "hook": "/usr/bin/nvidia-container-runtime-hook",
   "arguments": ["prestart"],
   "annotations": ["sandbox"],
   "stage": [ "prestart" ]
}


# yum install -y podman

# podman run --user 1000:1000 --security-opt=no-new-privileges --cap-drop=ALL \
--security-opt label=type:nvidia_container_t \
docker.io/mirrorgooglecontainers/cuda-vector-add:v0.1
[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done
```

