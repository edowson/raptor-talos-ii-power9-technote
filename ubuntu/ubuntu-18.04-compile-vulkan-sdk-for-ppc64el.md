# Ubuntu-18.04 - Compile Vulkan SDK for `ppc64el`

## Overview

This document describes how to compile the Vulkan SDK for `ppc64el` host architecture.

## Procedure

### Step 01.00: Install Vulkan SDK.

#### Step 01.01: Compile Vulkan SDK for `ppc64el`.

**Note:** While the procedure outlined below is generally correct, the current version of the
NVIDIA drivers (418.67) for `ppc64el` doesn't have support for Vulkan rendering. You will have
to wait for a newer driver (e.g. 430.40 or higher).

Download the vulkan sdk:
```bash
VULKAN_SDK_VERSION="1.1.114.0"
echo "downloading Vulkan SDK $VULKAN_SDK_VERSION" \
&& wget -q --show-progress --progress=bar:force:noscroll https://sdk.lunarg.com/sdk/download/$VULKAN_SDK_VERSION/linux/vulkansdk-linux-x86_64-$VULKAN_SDK_VERSION.tar.gz?Human=true -O /tmp/vulkansdk-linux-x86_64-$VULKAN_SDK_VERSION.tar.gz
```

Extract the vulkan sdk:
```bash
echo "extracting Vulkan SDK $VULKAN_SDK_VERSION" \
&& sudo mkdir -p /project/software/library/vulkan \
&& sudo chown -R administrator:administrator /project \
&& tar -xf /tmp/vulkansdk-linux-x86_64-$VULKAN_SDK_VERSION.tar.gz -C /project/software/library/vulkan \
&& rm /tmp/vulkansdk-linux-x86_64-$VULKAN_SDK_VERSION.tar.gz
```

The root SDK directory contains a script named vulkansdk to aid with building binaries. This script will build all the binaries included in the SDK, and also build those that are not included for compatibility reasons.

Modify the sources in the vulkan sdk to build for `ppc64el`:
```bash
nano /project/software/library/vulkan/1.1.114.0/vulkansdk
```

Make the following changes to the `vulkansdk` file, line 196 to query the host system architecture and set the `$ARCHDIR` output folder:
```bash
ARCH=`dpkg --print-architecture`
ARCHDIR="$SDKDIR/$ARCH"
```

Install required packages:
```bash
sudo apt-get install --no-install-recommends -q -y \
bison \
cmake cmake-curses-gui \
g++ \
gcc \
git \
libglm-dev \
libpng-dev \
libmirclient-dev \
libpciaccess0 \
libpython3.6 \
libwayland-dev \
libx11-dev \
libxrandr-dev \
libxcb-dri3-0 \
libxcb-dri3-dev
libxcb-ewmh-dev \
libxcb-keysyms1-dev \
libxcb-present0 \
libxcb-randr0-dev

# python3 packages
sudo apt-get install \
python-minimal \
python3-distutils
```


Setup `~/.bashrc` for vulkan headers:
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
export CPU_ARCHITECTURE=$(uname -m)

# Vulkan
VULKAN_SDK_VERSION="1.1.114.0"
export VULKAN_SDK="/project/software/library/vulkan/${VULKAN_SDK_VERSION}/$CPU_ARCHITECTURE"
export VK_LAYER_PATH="${VULKAN_SDK}/etc/explicit_layer.d"

export LD_LIBRARY_PATH="${VULKAN_SDK}/lib${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}"
export PATH="${VULKAN_SDK}/bin${PATH:+:${PATH}}"
```

Source `~/.bashrc` to set environment variable before building the vulkan sdk:
```bash
source ~/.bashrc
```

Build the vulkan sdk tools:
```bash
cd /project/software/library/vulkan/${VULKAN_SDK_VERSION}

bash vulkansdk glslang loader layers tools lunarg-tools \
shaderc spirvtools spirvcross
```


**Note:** The vulkan layer_factory [sources](https://github.com/LunarG/VulkanTools/tree/master/layer_factory)
need to be patched to remove hard-coded entries for `x86_64` with `ppc64el` in the `sources/layer_factor/CMakeLists.txt` file:
```bash
sed -i 's/x86_64/ppc64el/g' "/project/software/library/vulkan/${VULKAN_SDK_VERSION}/source/layer_factory/CMakeLists.txt"
```

Build vulkan layer_factory:
```bash
bash vulkansdk vlf
```

Note: The `VulkanSamples` [sources](https://github.com/LunarG/VulkanSamples)
need to be patched to remove hard-coded entries for `x86_64` with `ppc64el` in the `samples/CMakeLists.txt` file:
```bash
sed -i 's/x86_64/ppc64el/g' "/project/software/library/vulkan/${VULKAN_SDK_VERSION}/samples/CMakeLists.txt"
```
Build the vulkan sdk samples:
```bash
bash vulkansdk samples
```

After successfully running vulkansdk the files will be located under the "ppc64el" directory.


####  Step 01.02: Setup nvidia opengl and vulkan driver icds.

Ensure that the glvnd driver icd exists.

```bash
sudo nano /usr/share/glvnd/egl_vendor.d/10_nvidia.json
```

Ensure that the following entries exist:
```json
{
    "file_format_version" : "1.0.0",
    "ICD" : {
        "library_path" : "libEGL_nvidia.so.0"
    }
}
```

Create that the vulkan driver icd.

Create the file

```bash
sudo mkdir -p /usr/share/vulkan/icd.d
sudo nano /usr/share/vulkan/icd.d/nvidia_icd.json
```

Add the following entries:
```json
{
    "file_format_version" : "1.0.0",
    "ICD" : {
        "library_path": "libGLX_nvidia.so.0",
        "api_version" : "1.1.95"
    }
}
```

Check the nvidia-driver api version:
```bash
vulkaninfo | grep apiVersion

'DISPLAY' environment variable not set... skipping surface info
	apiVersion     = 0x40105f  (1.1.95)
```

Check vulkan ray-tracing support:
```bash
vulkaninfo | grep VK_NV_ray_tracing
```
