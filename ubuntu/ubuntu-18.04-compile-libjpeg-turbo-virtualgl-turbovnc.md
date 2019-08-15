# Ubuntu-18.04 - Configure VirtualGL and TurboVNC

This document describes how to configure VirtualGL and TurboVNC from sources.

## Procedure

### Step 01.00: Build libjpeg-turbo.

When building libjpeg-turbo from sources, you will need to additionally build
the `nasm` library for `x86_64` architecture. You can skip building `nasm` for `ppc64le` architecture.

#### Step 01.01: [x86_64 client only] Build nasm library from sources for x86_64 libmirclient.

We will need to additionally download and build the `nasm` library for `x86_64` clients.

Build the nasm library from sources for `x86_64` clients. **Skip this step for the `ppc64le` remote server**:
```bash
wget http://www.nasm.us/pub/nasm/releasebuilds/2.14/nasm-2.14.tar.bz2
tar -xvjf nasm-2.14.tar.bz2
cd nasm-2.14
./configure
make  -j8
sudo make install
```

#### Step 01.02: Build libjepg-turbo from sources.

Install required packages:
```bash
sudo apt install \
default-jdk
```

Clone sources:
```bash
cd /project/software/library
git clone https://github.com/libjpeg-turbo/libjpeg-turbo.git libjpeg-turbo/src
cd libjpeg-turbo/src

# checkout 2.0.2 tag and create a local branch
git checkout 2.0.2
git checkout -b 2.0.2
```

Configure:
```bash
cd /project/software/library/libjpeg-turbo
mkdir build; cd build;

ccmake \
-DCMAKE_INSTALL_PREFIX="/opt/libjpeg-turbo" \
-DCMAKE_BUILD_TYPE="Release" \
-DWITH_JAVA="ON" \
../src
```

Build:
```bash
make -j8
```

Install:
```bash
sudo make install
```

**Notes:**
01. The latest version of libjpeg-turbo is 2.0.2.
    [github.com/libjpeg-turbo/libjpeg-turbo](https://github.com/libjpeg-turbo/libjpeg-turbo)


### Step 02.00: Build virtualgl.

#### Step 02.01: Build virtualgl from sources.

Install required packages:
```bash
sudo apt install \
libgles2-mesa-dev \
libglu1-mesa-dev \
libssl-dev \
libxcb-keysyms1-dev \
libxcomposite-dev \
libxcursor-dev \
libxft-dev \
libxinerama-dev \
libxkbfile-dev \
libxmu-dev \
libxpm-dev \
libxres-dev \
libxtst-dev \
libxv-dev \
libxt-dev
```

Clone sources:
```bash
cd /project/software/library
git clone https://github.com/VirtualGL/virtualgl.git virtualgl/src
cd virtualgl/src

# checkout master branch
git checkout master

# checkout 2.6.2 tag and create a local branch
#git checkout 2.6.2
#git checkout -b 2.6.2
```

Configure:
```bash
cd /project/software/library/virtualgl
mkdir build; cd build;

ccmake \
-DCMAKE_INSTALL_PREFIX="/opt/VirtualGL" \
-DCMAKE_BUILD_TYPE="Release" \
-DTJPEG_INCLUDE_DIR="/opt/libjpeg-turbo/include" \
-DTJPEG_LIBRARY="-L/opt/libjpeg-turbo/lib64 -lturbojpeg" \
-DVGL_USESSL=1 \
-DVGL_SYSTEMGLX="ON" \
-DVGL_SYSTEMXCB="ON" \
-DVGL_USEIFR="OFF" \
../src
```

Fix broken link to `/usr/lib/powerpc64le-linux-gnu/libGL.so`. While build, you might
encounter the following error:
```bash
[  8%] Building CXX object util/CMakeFiles/glreadtest.dir/glreadtest.cpp.o
make[2]: *** No rule to make target '/usr/lib/powerpc64le-linux-gnu/libGL.so', needed by 'bin/glreadtest'.  Stop.
CMakeFiles/Makefile2:708: recipe for target 'util/CMakeFiles/glreadtest.dir/all' failed
```

To fix this, uodate the symbolic link:
```bash
cd /usr/lib/powerpc64le-linux-gnu/
ls -la libGL.*

# libGL.so is symlinked to non-existent libGL.so.1.0.0
lrwxrwxrwx 1 root root      14 May 10 08:17 libGL.so -> libGL.so.1.0.0
lrwxrwxrwx 1 root root      14 Aug 11 02:52 libGL.so.1 -> libGL.so.1.7.0
-rwxr-xr-x 1 root root 1437056 Aug 11 02:52 libGL.so.1.7.0

# fix broken libGL.so symbolic link
sudo rm /usr/lib/powerpc64le-linux-gnu/libGL.so
cd /usr/lib/powerpc64le-linux-gnu/
sudo ln -s libGL.so.1 libGL.so
```

Build:
```bash
cd /project/software/library/virtualgl/build
make -j8
```

Install:
```bash
sudo make install
```

Create debian style binary package:
```bash
make deb
```

**Note:**
01. The latest stable release of libssl-dev is 1.1.1.
    [openssl project](https://www.openssl.org/source/)
    [openssl-1.1.1c.tar.gz](https://www.openssl.org/source/openssl-1.1.1c.tar.gz)
    [github.com/openssl/openssl](https://github.com/openssl/openssl)


### Step 03.00: Build turbovnc.

#### Step 03.01: Build turbovnc from sources.

Install required packages:
```bash
sudo apt install \
libpam0g-dev \
libxfont-dev
```

Clone sources:
```bash
cd /project/software/library
git clone https://github.com/TurboVNC/turbovnc.git turbovnc/src
cd turbovnc/src

# checkout master branch
git checkout master

# checkout 2.2.2 tag and create a local branch
#git checkout 2.2.2
#git checkout -b 2.2.2
```

Configure:
```bash
cd /project/software/library/turbovnc
mkdir build; cd build;

ccmake \
-DCMAKE_INSTALL_PREFIX="/opt/TurboVNC" \
-DCMAKE_BUILD_TYPE="Release" \
-DTJPEG_INCLUDE_DIR="/opt/libjpeg-turbo/include" \
-DTJPEG_LIBRARY="-L/opt/libjpeg-turbo/lib64 -lturbojpeg" \
-DTVNC_BUILDJAVA=1 \
-DTVNC_GLX="ON" \
../src
```

Build:
```bash
cd /project/software/library/turbovnc/build
make -j8
```

Install:
```bash
sudo make install
```

---

## Issues

01. [Compiler error on Ubuntu Linux with Nvidia Drivers: No rule to make target `/usr/lib/x86_64-linux-gnu/libGL.so' #2087 - RobotLocomotion/drake](https://github.com/RobotLocomotion/drake/issues/2087)

02. [Not creating vgl_xauth_key when using gdm3 in ubuntu server #65](https://github.com/VirtualGL/virtualgl/issues/65)

03. [gdm does not run /etc/gdm/*/Default scripts properly!](https://gitlab.gnome.org/GNOME/gdm/issues/317)

---

## Repositories

01. [github.com/openssl/openssl](https://github.com/openssl/openssl)
> TLS/SSL and crypto library.

---

## Technotes

### Ubuntu

01. [How to install the GNOME Desktop on Ubuntu Server 18.04 - TechRepublic](https://www.techrepublic.com/article/how-to-install-the-gnome-desktop-on-ubuntu-server-18-04/)

02. [Install GUI on Ubuntu Server 18.04 Bionic Beaver - Lubos Rendek -  LinuxConfig](https://linuxconfig.org/install-gui-on-ubuntu-server-18-04-bionic-beaver)

03. [How to Remove or Delete PPA in Ubuntu - It's FOSS](https://itsfoss.com/how-to-remove-or-delete-ppas-quick-tip/)

### libjpeg-turbo

01. [Compile and use libjpeg-turbo on ubuntu 16.04](https://kezunlin.me/post/9f626e7a/)

### VirtualGL

01. [Headless nVidia Mini How-To - VirtualGL](https://virtualgl.org/Documentation/HeadlessNV)

02. [How to set up VirtualGL and TurboVNC for use with ParaViewWeb - Kitware](https://kitware.github.io/paraviewweb/docs/virtualgl_turbovnc_howto.html)

03. [Remote accelerated graphics with VirtualGL and TurboVNC - Jan Hreha - 20150804](https://summerofhpc.prace-ri.eu/remote-accelerated-graphics-with-virtualgl-and-turbovnc/)

---

## Related Topics

### Ubuntu

01. [How to create a bootable Ubuntu USB flash drive from terminal? - Ubuntu](https://askubuntu.com/questions/372607/how-to-create-a-bootable-ubuntu-usb-flash-drive-from-terminal)

## EGL

01. [How do I get EGL and OpenGLES libraries for Ubuntu running on VirtualBox?](https://askubuntu.com/questions/244133/how-do-i-get-egl-and-opengles-libraries-for-ubuntu-running-on-virtualbox/821812)

## X11

01. X11 connection rejected because of wrong authentication [](https://community.hpe.com/t5/Networking/X11-connection-rejected-because-of-wrong-authentication/td-p/6987197#.XVLrQXUzbds)

Solution:
Set `X11UseLocalhost` variable to `no` in the file `/etc/opt/ssh/sshd_config`

## VirtualGL

01.[vgl transport: '[VGL] ERROR: Could not open display :0.'](https://virtualgl-users.narkive.com/kUckmG7X/vgl-transport-vgl-error-could-not-open-display-0)
