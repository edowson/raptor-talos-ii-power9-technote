# Ubuntu-18.04 Installation Guide for IBM Power9 - Raptor Talos II Secure Workstation - Install NVIDIA proprietary graphics drivers using standalone installer

## Overview

This document describes the setup procedure for installing NVIDIA's proprietary graphcis drivers on the Talos II secure workstation.

## Procedure

### Step 01.00: Install NVIDIA graphics drivers.

#### Step 01.01: Blacklist nouveau drivers.

Ensure you are using only Nvidia proprietary drivers by blacklisting `Nouveau`, Ubuntu's built-in Open Source driver.

Create blacklist-nouveau.conf
```bash
sudo nano /etc/modprobe.d/blacklist-nouveau.conf
```

Include the following:
```bash
blacklist nouveau
options nouveau modeset=0
```

Enter the following linux command to regenerate initramfs:
```bash
sudo update-initramfs -u
```

Reboot your system:
```bash
sudo reboot -i NOW
```

####  Step 01.02: Install NVIDIA proprietary graphics drivers.

List installed PCI devices:
```bash
sudo lspci

0000:00:00.0 PCI bridge: IBM Device 04c1
0001:00:00.0 PCI bridge: IBM Device 04c1
0001:01:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM961/PM961
0002:00:00.0 PCI bridge: IBM Device 04c1
0003:00:00.0 PCI bridge: IBM Device 04c1
0003:01:00.0 USB controller: Texas Instruments TUSB73x0 SuperSpeed USB 3.0 xHCI Host Controller (rev 02)
0004:00:00.0 PCI bridge: IBM Device 04c1
0004:01:00.0 Ethernet controller: Broadcom Inc. and subsidiaries NetXtreme BCM5719 Gigabit Ethernet PCIe (rev 01)
0004:01:00.1 Ethernet controller: Broadcom Inc. and subsidiaries NetXtreme BCM5719 Gigabit Ethernet PCIe (rev 01)
0005:00:00.0 PCI bridge: IBM Device 04c1
0005:01:00.0 PCI bridge: ASPEED Technology, Inc. AST1150 PCI-to-PCI Bridge (rev 04)
0005:02:00.0 VGA compatible controller: ASPEED Technology, Inc. ASPEED Graphics Family (rev 41)
0030:00:00.0 PCI bridge: IBM Device 04c1
0030:01:00.0 3D controller: NVIDIA Corporation GV100GL [Tesla V100 PCIe 16GB] (rev a1)
0031:00:00.0 PCI bridge: IBM Device 04c1
0032:00:00.0 PCI bridge: IBM Device 04c1
0033:00:00.0 PCI bridge: IBM Device 04c1
```

We can see that the NVIDIA V100 GPU is connect to PCIe slot `0030:01:00.0`.

Download the driver:
```bash
NVIDIA_DRIVER_VERSION='418.67'
NVIDIA_DRIVER_RELEASE_DATE='2019.5.7'
OS_DISTRO='ubuntu'
OS_VERSION='1804'
ARCH='ppc64le'

URL1="http://us.download.nvidia.com/tesla/$NVIDIA_DRIVER_VERSION/NVIDIA-Linux-$ARCH-$NVIDIA_DRIVER_VERSION.run"
URL2="http://us.download.nvidia.com/tesla/$NVIDIA_DRIVER_VERSION/nvidia-driver-local-repo-$OS_DISTRO$OS_VERSION-${NVIDIA_DRIVER_VERSION}_1.0-1_$ARCH.deb"

wget -q --show-progress --progress=bar:force:noscroll http://us.download.nvidia.com/tesla/$NVIDIA_DRIVER_VERSION/NVIDIA-Linux-$ARCH-$NVIDIA_DRIVER_VERSION.run -O /tmp/NVIDIA-Linux-$ARCH-$NVIDIA_DRIVER_VERSION.run
```

Install the driver, with `dkms` support and overwrite existing `libglvnd` files.
```bash
sudo bash /tmp/NVIDIA-Linux-$ARCH-$NVIDIA_DRIVER_VERSION.run
```

To extract the contents of the nvidia-driver:
```bash
bash /tmp/NVIDIA-Linux-$ARCH-$NVIDIA_DRIVER_VERSION.run --extract-only
```

####  Step 01.03: Verify nvidia driver installation.

Run `nvidia-smi`:
```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.67.00    Driver Version: 418.67.00    CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla V100-PCIE...  On   | 00000030:01:00.0 Off |                    0 |
| N/A   35C    P0    26W / 250W |      0MiB / 16130MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

Check installed libraries.

```bash
cd /usr/lib/powerpc64le-linux-gnu/

# nvidia libraries
ls -la libnvidia*

lrwxrwxrwx 1 root root       18 Aug 11 02:52 libnvidia-cfg.so -> libnvidia-cfg.so.1
lrwxrwxrwx 1 root root       23 Aug 11 02:52 libnvidia-cfg.so.1 -> libnvidia-cfg.so.418.67
-rwxr-xr-x 1 root root   207904 Aug 11 02:52 libnvidia-cfg.so.418.67
-rwxr-xr-x 1 root root 25914464 Aug 11 02:52 libnvidia-eglcore.so.418.67
lrwxrwxrwx 1 root root       30 Aug 11 02:52 libnvidia-egl-wayland.so.1 -> libnvidia-egl-wayland.so.1.1.2
-rwxr-xr-x 1 root root    44056 Aug 11 02:52 libnvidia-egl-wayland.so.1.1.2
lrwxrwxrwx 1 root root       21 Aug 11 02:52 libnvidia-encode.so -> libnvidia-encode.so.1
lrwxrwxrwx 1 root root       26 Aug 11 02:52 libnvidia-encode.so.1 -> libnvidia-encode.so.418.67
-rwxr-xr-x 1 root root   155400 Aug 11 02:52 libnvidia-encode.so.418.67
-rwxr-xr-x 1 root root   334904 Aug 11 02:52 libnvidia-fatbinaryloader.so.418.67
-rwxr-xr-x 1 root root 26397320 Aug 11 02:52 libnvidia-glcore.so.418.67
-rwxr-xr-x 1 root root   677904 Aug 11 02:52 libnvidia-glsi.so.418.67
-rwxr-xr-x 1 root root 14122992 Aug 11 02:52 libnvidia-glvkspirv.so.418.67
-rwxr-xr-x 1 root root  1574264 Aug 11 02:52 libnvidia-gtk2.so.418.67
lrwxrwxrwx 1 root root       17 Aug 11 02:52 libnvidia-ml.so -> libnvidia-ml.so.1
lrwxrwxrwx 1 root root       22 Aug 11 02:52 libnvidia-ml.so.1 -> libnvidia-ml.so.418.67
-rwxr-xr-x 1 root root  1577472 Aug 11 02:52 libnvidia-ml.so.418.67
lrwxrwxrwx 1 root root       26 Aug 11 02:52 libnvidia-opencl.so.1 -> libnvidia-opencl.so.418.67
-rwxr-xr-x 1 root root 29180824 Aug 11 02:52 libnvidia-opencl.so.418.67
lrwxrwxrwx 1 root root       26 Aug 11 02:52 libnvidia-opticalflow.so -> libnvidia-opticalflow.so.1
lrwxrwxrwx 1 root root       31 Aug 11 02:52 libnvidia-opticalflow.so.1 -> libnvidia-opticalflow.so.418.67
-rwxr-xr-x 1 root root    97824 Aug 11 02:52 libnvidia-opticalflow.so.418.67
lrwxrwxrwx 1 root root       29 Aug 11 02:52 libnvidia-ptxjitcompiler.so -> libnvidia-ptxjitcompiler.so.1
lrwxrwxrwx 1 root root       34 Aug 11 02:52 libnvidia-ptxjitcompiler.so.1 -> libnvidia-ptxjitcompiler.so.418.67
-rwxr-xr-x 1 root root  8039952 Aug 11 02:52 libnvidia-ptxjitcompiler.so.418.67
-rwxr-xr-x 1 root root     5264 Aug 11 02:52 libnvidia-tls.so.418.67


# egl libraries
ls -la libEGL*

lrwxrwxrwx 1 root root      20 Jul 18 11:44 libEGL_mesa.so.0 -> libEGL_mesa.so.0.0.0
-rw-r--r-- 1 root root  398384 Jul 18 11:44 libEGL_mesa.so.0.0.0
lrwxrwxrwx 1 root root      23 Aug 11 02:52 libEGL_nvidia.so.0 -> libEGL_nvidia.so.418.67
-rwxr-xr-x 1 root root 1266168 Aug 11 02:52 libEGL_nvidia.so.418.67
lrwxrwxrwx 1 root root      11 Aug 11 02:52 libEGL.so -> libEGL.so.1
lrwxrwxrwx 1 root root      15 Aug 11 02:52 libEGL.so.1 -> libEGL.so.1.1.0
-rwxr-xr-x 1 root root   86944 Aug 11 02:52 libEGL.so.1.1.0

# glx libraries
ls -la libGLX*

lrwxrwxrwx 1 root root      23 Aug 11 02:52 libGLX_indirect.so.0 -> libGLX_nvidia.so.418.67
lrwxrwxrwx 1 root root      20 Jul 18 11:44 libGLX_mesa.so.0 -> libGLX_mesa.so.0.0.0
-rw-r--r-- 1 root root  725688 Jul 18 11:44 libGLX_mesa.so.0.0.0
lrwxrwxrwx 1 root root      23 Aug 11 02:52 libGLX_nvidia.so.0 -> libGLX_nvidia.so.418.67
-rwxr-xr-x 1 root root 1661296 Aug 11 02:52 libGLX_nvidia.so.418.67
lrwxrwxrwx 1 root root      11 Aug 11 02:52 libGLX.so -> libGLX.so.0
-rwxr-xr-x 1 root root   80472 Aug 11 02:52 libGLX.so.0
```
