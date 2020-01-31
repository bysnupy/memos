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

# cat <<EOF > /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json
{
   "hook": "/usr/bin/nvidia-container-runtime-hook",
   "arguments": ["prestart"],
   "annotations": ["sandbox"],
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

# oc label node ip-10-0-21-68.ap-northeast-1.compute.internal openshift.com/gpu-accelerator=true

# git clone https://github.com/redhat-performance/openshift-psap.git
# cd openshift-psap/blog/gpu/device-plugin
# oc create -f nvidia-device-plugin.yml

# oc get pods -n kube-system
NAME                                                                READY     STATUS    RESTARTS   AGE
master-api-ip-10-0-21-68.ap-northeast-1.compute.internal           1/1       Running   0          10h
master-controllers-ip-10-0-21-68.ap-northeast-1.compute.internal   1/1       Running   0          10h
master-etcd-ip-10-0-21-68.ap-northeast-1.compute.internal          1/1       Running   0          10h
nvidia-device-plugin-daemonset-hscq6                                1/1       Running   0          1m

# oc logs -n kube-system nvidia-device-plugin-daemonset-hscq6
2020/01/31 02:43:31 Loading NVML
2020/01/31 02:43:31 Failed to initialize NVML: could not load NVML library.
2020/01/31 02:43:31 If this is a GPU node, did you set the docker default runtime to `nvidia`?
2020/01/31 02:43:31 You can check the prerequisites at: https://github.com/NVIDIA/k8s-device-plugin#prerequisites
2020/01/31 02:43:31 You can learn how to set the runtime at: https://github.com/NVIDIA/k8s-device-plugin#quick-start

# oc describe node | grep -E 'Capacity|Allocatable|gpu'
                    openshift.com/gpu-accelerator=true
Capacity:
 nvidia.com/gpu:  1
Allocatable:
 nvidia.com/gpu:  1
  nvidia.com/gpu  0            0

# oc logs nvidia-device-plugin-daemonset-hjll6
2020/01/31 02:50:43 Loading NVML
2020/01/31 02:50:43 Fetching devices.
2020/01/31 02:50:43 Starting FS watcher.
2020/01/31 02:50:43 Starting OS watcher.
2020/01/31 02:50:43 Could not start device plugin: listen unix /var/lib/kubelet/device-plugins/nvidia.sock: bind: permission denied
2020/01/31 02:50:43 Could not contact Kubelet, retrying. Did you enable the device plugin feature gate?
2020/01/31 02:50:43 You can check the prerequisites at: https://github.com/NVIDIA/k8s-device-plugin#prerequisites
2020/01/31 02:50:43 You can learn how to set the runtime at: https://github.com/NVIDIA/k8s-device-plugin#quick-start

# restorecon -Rv /var/lib/kubelet
restorecon reset /var/lib/kubelet context system_u:object_r:var_lib_t:s0->system_u:object_r:container_file_t:s0
restorecon reset /var/lib/kubelet/device-plugins context system_u:object_r:var_lib_t:s0->system_u:object_r:container_file_t:s0
restorecon reset /var/lib/kubelet/device-plugins/kubelet_internal_checkpoint context system_u:object_r:var_lib_t:s0->system_u:object_r:container_file_t:s0
restorecon reset /var/lib/kubelet/device-plugins/kubelet.sock context system_u:object_r:var_lib_t:s0->system_u:object_r:container_file_t:s0


# cat /usr/share/containers/oci/hooks.d/oci-nvidia-hook.json
{
   "cmd": [".*"],
   "hook": "/usr/bin/nvidia-container-runtime-hook",
   "arguments": ["prestart"],
   "stage": [ "prestart" ]
}

```

