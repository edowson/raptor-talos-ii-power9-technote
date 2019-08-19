# Ubuntu-18.04 Installation Guide for IBM Power9 - Raptor Talos II Secure Workstation - Install NVIDIA Graphics, CUDA, cuDNN and TensorRT libraries

## Overview

This document describes the setup procedure for installing NVIDIA Graphics, CUDA, cuDNN and TensorRT libraries on the Talos II secure workstation.

## Procedure

### Step 01.00: Install NVIDIA Graphics and acceleration libraries.

#### Step 01.01: Setup network repo.

```bash
OS_DISTRO="ubuntu"
OS_VERSION="1804"
OS_ARCH="ppc64el"

# package versions
CUDA_VERSION="10.1"
CUDA_MAJOR_VERSION=`echo $CUDA_VERSION | cut -d. -f1`
CUDA_MINOR_VERSION=`echo $CUDA_VERSION | cut -d. -f2`
CUDNN_VERSION="7.6.2.24"
CUDNN_MAJOR_VERSION=`echo $CUDNN_VERSION | cut -d. -f1`
NCCL2_VERSION="2.4.7"
TENSORRT_VERSION="5.1.5.0"
```

Install required packages:
```bash
sudo apt install \
apt-transport-https \
ca-certificates \
curl \
gnupg2
```

Use the network repo for install cuda on Ubuntu-18.04:
```bash
# setup keys: nvidia compute/cuda for ubuntu-18.04/ppc64el
echo "importing gpg keys for nvidia compute/cuda $OS_DISTRO$OS_VERSION/$OS_ARCH apt repository"
curl -fsSL "https://developer.download.nvidia.com/compute/cuda/repos/$OS_DISTRO$OS_VERSION/$OS_ARCH/7fa2af80.pub" | sudo apt-key add -
wget -qO - "https://developer.download.nvidia.com/compute/cuda/repos/$OS_DISTRO$OS_VERSION/$OS_ARCH/7fa2af80.pub" | sudo apt-key add -

# setup sources.list
echo "deb https://developer.download.nvidia.com/compute/cuda/repos/$OS_DISTRO$OS_VERSION/$OS_ARCH /" \
| sudo tee /etc/apt/sources.list.d/cuda.list
echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/$OS_DISTRO$OS_VERSION/$OS_ARCH /" \
| sudo tee /etc/apt/sources.list.d/nvidia-ml.list
sudo apt update
```

#### Step 01.02: Install NVIDIA CUDA.

```bash
# meta-package
sudo apt install cuda-$CUDA_MAJOR_VERSION-$CUDA_MINOR_VERSION

# output:
The following additional packages will be installed:
  cuda-command-line-tools-10-1 cuda-compiler-10-1 cuda-cudart-10-1 cuda-cudart-dev-10-1 cuda-cufft-10-1
  cuda-cufft-dev-10-1 cuda-cuobjdump-10-1 cuda-cupti-10-1 cuda-curand-10-1 cuda-curand-dev-10-1 cuda-cusolver-10-1
  cuda-cusolver-dev-10-1 cuda-cusparse-10-1 cuda-cusparse-dev-10-1 cuda-documentation-10-1 cuda-driver-dev-10-1
  cuda-drivers cuda-gdb-10-1 cuda-gpu-library-advisor-10-1 cuda-libraries-10-1 cuda-libraries-dev-10-1
  cuda-license-10-1 cuda-memcheck-10-1 cuda-misc-headers-10-1 cuda-npp-10-1 cuda-npp-dev-10-1 cuda-nsight-10-1
  cuda-nsight-compute-10-1 cuda-nvcc-10-1 cuda-nvdisasm-10-1 cuda-nvgraph-10-1 cuda-nvgraph-dev-10-1 cuda-nvjpeg-10-1
  cuda-nvjpeg-dev-10-1 cuda-nvml-dev-10-1 cuda-nvprof-10-1 cuda-nvprune-10-1 cuda-nvrtc-10-1 cuda-nvrtc-dev-10-1
  cuda-nvtx-10-1 cuda-nvvp-10-1 cuda-runtime-10-1 cuda-samples-10-1 cuda-toolkit-10-1 cuda-tools-10-1
  cuda-visual-tools-10-1 libcublas-dev libcublas10 libnvidia-cfg1-418 libnvidia-common-418 libnvidia-common-430
  libnvidia-compute-418 libnvidia-decode-418 libnvidia-encode-418 libnvidia-fbc1-418 libnvidia-gl-418
  libnvidia-ifr1-418 libvdpau1 libxnvctrl0 mesa-vdpau-drivers nsight-compute-2019.4.0 nvidia-compute-utils-418
  nvidia-dkms-418 nvidia-driver-418 nvidia-kernel-common-418 nvidia-kernel-source-418 nvidia-modprobe nvidia-prime
  nvidia-settings nvidia-utils-418 screen-resolution-extra vdpau-driver-all xserver-xorg-video-nvidia-418
Suggested packages:
  libvdpau-va-gl1
The following NEW packages will be installed:
  cuda-10-1 cuda-command-line-tools-10-1 cuda-compiler-10-1 cuda-cudart-10-1 cuda-cudart-dev-10-1 cuda-cufft-10-1
  cuda-cufft-dev-10-1 cuda-cuobjdump-10-1 cuda-cupti-10-1 cuda-curand-10-1 cuda-curand-dev-10-1 cuda-cusolver-10-1
  cuda-cusolver-dev-10-1 cuda-cusparse-10-1 cuda-cusparse-dev-10-1 cuda-documentation-10-1 cuda-driver-dev-10-1
  cuda-drivers cuda-gdb-10-1 cuda-gpu-library-advisor-10-1 cuda-libraries-10-1 cuda-libraries-dev-10-1
  cuda-license-10-1 cuda-memcheck-10-1 cuda-misc-headers-10-1 cuda-npp-10-1 cuda-npp-dev-10-1 cuda-nsight-10-1
  cuda-nsight-compute-10-1 cuda-nvcc-10-1 cuda-nvdisasm-10-1 cuda-nvgraph-10-1 cuda-nvgraph-dev-10-1 cuda-nvjpeg-10-1
  cuda-nvjpeg-dev-10-1 cuda-nvml-dev-10-1 cuda-nvprof-10-1 cuda-nvprune-10-1 cuda-nvrtc-10-1 cuda-nvrtc-dev-10-1
  cuda-nvtx-10-1 cuda-nvvp-10-1 cuda-runtime-10-1 cuda-samples-10-1 cuda-toolkit-10-1 cuda-tools-10-1
  cuda-visual-tools-10-1 libcublas-dev libcublas10 libnvidia-cfg1-418 libnvidia-common-418 libnvidia-common-430
  libnvidia-compute-418 libnvidia-decode-418 libnvidia-encode-418 libnvidia-fbc1-418 libnvidia-gl-418
  libnvidia-ifr1-418 libvdpau1 libxnvctrl0 mesa-vdpau-drivers nsight-compute-2019.4.0 nvidia-compute-utils-418
  nvidia-dkms-418 nvidia-driver-418 nvidia-kernel-common-418 nvidia-kernel-source-418 nvidia-modprobe nvidia-prime
  nvidia-settings nvidia-utils-418 screen-resolution-extra vdpau-driver-all xserver-xorg-video-nvidia-418
```

#### Step 01.03: Power9 specific post-install setup.

Because of the addition of new features specific to the NVIDIA POWER9 CUDA driver, there are some additional setup requirements in order for the driver to function properly. These additional steps are not handled by the installation of CUDA packages, and failure to ensure these extra requirements are met will result in a non-functional CUDA driver installation.
There are two changes that need to be made manually after installing the NVIDIA CUDA driver to ensure proper operation:

The NVIDIA Persistence Daemon should be automatically started for POWER9 installations. Check that it is running with the following command:
```bash
$ systemctl status nvidia-persistenced
```

If it is not active, run the following command:
```bash
$ sudo systemctl enable nvidia-persistenced
```

Disable a udev rule installed by default in some Linux distributions that cause hot-pluggable memory to be automatically onlined when it is physically probed. This behavior prevents NVIDIA software from bringing NVIDIA device memory online with non-default settings. This udev rule must be disabled in order for the NVIDIA CUDA driver to function properly on POWER9 systems.

On Ubuntu 18.04, this rule can be found in:
```bash
/lib/udev/rules.d/40-vm-hotadd.rules
```
The rule generally takes a form where it detects the addition of a memory block and changes the 'state' attribute to online.

For example, in Ubuntu 18.04, the rule looks like this:
```bash
# Memory hotadd request
SUBSYSTEM=="memory", ACTION=="add", DEVPATH=="/devices/system/memory/memory[0-9]*", TEST=="state", ATTR{state}="online"
```

This rule must be disabled by copying the file from `/lib/udev/rules.d/` to `/etc/udev/rules.d` and commenting out, removing, or changing the hot-pluggable memory rule in the `/etc/udev/rules.d` copy so that it does not apply to POWER9 NVIDIA systems.

For example, in Ubuntu 18.04:
```bash
$ sudo cp /lib/udev/rules.d/40-vm-hotadd.rules /etc/udev/rules.d
$ sudo sed -i '/SUBSYSTEM=="memory", ACTION=="add"/d' /etc/udev/rules.d/40-vm-hotadd.rules
```


You will need to reboot the system to initialize the above changes.
```bash
sudo reboot -i NOW
```

#### Step 01.04: Update .bashrc.

Edit `~/.bashrc`:
```bash
nano ~/.bashrc
```

```bash
# Set locate
export LC_ALL="en_US.UTF-8"
export LANG="en_US.UTF-8"

# shell
export PS1="${debian_chroot:+($debian_chroot)}\u:\W\$ "

# host
export DPKG_ARCH=`dpkg --print-architecture`

# NVIDIA CUDA environment variables
CUDA_VERSION=10.1
CUDNN_VERSION=7.6.2.24
NCCL_VERSION=2.4.7-1
CUDA=/usr/local/cuda-${CUDA_VERSION}

## NVIDIA TensorRT environment variables
TENSORRT_VERSION=5.1.5.0
TENSORRT=/project/software/library/TensorRT-$TENSORRT_VERSION

# VirtualGL, TurboVNC
LIBTURBO_JPEG_DIR="/opt/libjpeg-turbo"
VIRTUALGL_DIR="/opt/VirtualGL"
TURBOVNC_DIR="/opt/TurboVNC"
export TVNC_MT=1
export TVNC_NTHREADS=32

# Vulkan
VULKAN_SDK_VERSION="1.1.114.0"
export VULKAN_SDK="/project/software/library/vulkan/${VULKAN_SDK_VERSION}/$DPKG_ARCH"
export VK_LAYER_PATH="${VULKAN_SDK}/etc/explicit_layer.d"

export LD_LIBRARY_PATH="${CUDA}/lib64:${CUDA}/extras/CUPTI/lib64:${TENSORRT}/lib:${VULKAN_SDK}/lib:${LIBTURBO_JPEG_DIR}/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}"
export PATH="${HOME}/bin:${CUDA}/bin:${VULKAN_SDK}/bin:${VIRTUALGL_DIR}/bin:${TURBOVNC_DIR}/bin${PATH:+:${PATH}}"
```

Source `~/.bashrc` to set environment variable:
```bash
source ~/.bashrc
```

---

## Repositories

01. [compute/cuda/repos/ubuntu1804/ppc64el](https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/ppc64el/)

02. [cuda/dist/ubuntu18.04/10.1/base/Dockerfile](https://gitlab.com/nvidia/cuda/blob/master/dist/ubuntu18.04/10.1/base/Dockerfile)

---

## Technotes

01. [Steps on how to setup and configure Volta GPUs with CUDA 10.x on Ubuntu 18.04.x running on IBM Power System AC922](https://developer.ibm.com/recipes/tutorials/steps-on-how-to-setup-and-configure-volta-gpus-with-cuda-10-x-on-ubuntu-18-04-x-running-on-ibm-power-system-ac922/)

---
