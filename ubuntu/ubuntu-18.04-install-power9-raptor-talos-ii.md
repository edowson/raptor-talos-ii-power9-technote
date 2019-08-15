# Ubuntu-18.04 Installation Guide for IBM Power9 - Raptor Talos II Secure Workstation

## Overview

This machine has a V100 GPU installed with 256GB RAM.

In terms of network planning, static IP addresses need to be assigned to:
- BMC: 192.168.1.10
- Host adapter 1: 192.168.1.11
- Host adapter 2: (leave unassigned)

## Procedure

### Step 01.00: Setup

#### Step 01.01: Download and install the OpenBMC tool.

Download the OpenBMC tool from this link:
[Scale-out LC System Event Log Collection Tool](http://www14.software.ibm.com/webapp/set2/sas/f/lopdiags/scaleOutLCdebugtool.html)

The IBM OpenBMC tool is used to interact with the new OpenBMC on the 8335-GTG, 8335-GTC, 8335-GTW, 8335-GTH, and 8335-GTX products.

Pre-requisites:
```
python 3
python 3 requests
```

For RHEL distributions:
[openbmctool-1.14-0.noarch.rpm](http://www14.software.ibm.com/webapp/set2/sas/f/lopdiags/data/openbmctool-1.14-0.noarch.rpm)

For other operating systems:
[openbmctool.zip](http://www14.software.ibm.com/webapp/set2/sas/f/lopdiags/data/openbmctool-1.14.zip)

Download the `openbmctool`:
```bash
wget -q --show-progress --progress=bar:force:noscroll http://www14.software.ibm.com/webapp/set2/sas/f/lopdiags/data/openbmctool-1.14.zip
```

Extract the OpenBMC tool:
```bash
# create directories
sudo mkdir -p /opt/ibm/openbmctool
mkdir ~/bin

# extract
unzip ~/Downloads/openbmctool-1.14.zip -d /opt/ibm/openbmctool

# create a link to the file
ln -s -f /opt/ibm/openbmctool/openbmctool.py ~/bin/openbmctool
```

Create a conda virtual environment:
```bash
conda create -n bmc36 python=3.6
conda activate bmc36

# install required python packages
conda install requests
```

Usage:
```bash
python openbmctool -H <bmc IP> or <bmc hostname> -U <username> -P <password> chassis power status
```


#### Step 01.02: Connect to remote system via BMC

Remote host information:
```bash
export BMC_IP="192.168.1.10"
export BMC_USER="root"
export BMC_PASSWORD="OpenBMC"

SSH RSA fingerprint: 5a:d2:5c:2e:a6:e7:c7:15:b7:d0:cb:ae:24:b7:32:1e
```

Check system status:
```bash
# connect
python openbmctool -H $BMC_IP -U $BMC_USER -P $BMC_PASSWORD chassis power on

Attempting login...
Chassis Power State: Off
Host Power State: Off
BMC Power State: Ready
User root has been logged out
```


Power on the system:
```bash
python openbmctool -H $BMC_IP -U $BMC_USER -P $BMC_PASSWORD chassis power on
```

Login to the BMC `petitboot` shell:
```bash
ssh $BMC_USER@$BMC_IP
root@$BMC_PASSWORD

# console
root@talos:~#

# check version
uname -a
Linux talos 5.0.7-a8a208fa7346ad643e8f6100c49cb7b8468b6d38 #1 Tue Apr 30 18:53:46 UTC 2019 armv6l GNU/Linux
```


#### Step 01.03: Download installation media.

The official Ubuntu images can be downloaded from `http://cdimage.ubuntu.com/releases/18.04.3/release/`


Type the following commands:
```bash
obmcutil poweron
obmc-console-client
```

This should print the following progress message:
```bash
--== Welcome to Hostboot hostboot-3beba24/hbicore.bin ==--

  3.09649|secure|SecureROM valid - enabling functionality
  5.51647|Booting from SBE side 0 on master proc=00050000
  5.56517|ISTEP  6. 5 - host_init_fsi
  5.87670|ISTEP  6. 6 - host_set_ipl_parms
  6.19806|ISTEP  6. 7 - host_discover_targets
...
```

To detatch from console, use the disconnect sequence `<ENTER> ~ .`


Download the installation media using `wget` from the `petitboot` host shell:
```bash
export OS_DISTRO="ubuntu"
export OS_VERSION="18.04.3"
export OS_ARCH="ppc64el"
export OS_IMAGE_TYPE="server"
export ISO_IMAGE="$OS_DISTRO-$OS_VERSION-$OS_IMAGE_TYPE-$OS_ARCH.iso"

# download the iso image
wget http://cdimage.ubuntu.com/releases/$OS_VERSION/release/$ISO_IMAGE -O /tmp/$ISO_IMAGE

# verify the checksum
sha256sum /tmp/$ISO_IMAGE

7497009cd83ba6958f0d2872f6a0c7fe7fcac343cce8ed3d444e691c80735095  /tmp/ubuntu-18.04.3-server-ppc64el.iso
```

Write it to `/dev/sda` with `dd`:
```bash
dd if=/tmp/$ISO_IMAGE of=/dev/sda bs=4M && sync
```

After the image has been written to `/dev/sda`, type `exit` to return to the `petitboot` screen
and select `Install Ubuntu Server`.

```
Petitboot (v1.10.3-pc52bacd)                         T2P9D01 REV 1.01 A1000362
──────────────────────────────────────────────────────────────────────────────
 [USB: sda / 2019-08-05-18-47-03-00]
   Rescue a broken system (HWE)
   Check disc for defects (HWE)
   Install MAAS Rack Controller (HWE)
   Install MAAS Region Controller (HWE)
   Install Ubuntu Server (HWE)
   Rescue a broken system
   Check disc for defects
   Install MAAS Rack Controller
   Install MAAS Region Controller
*  Install Ubuntu Server

 System information
 System configuration
 System status log
 Language
 Rescan devices
 Retrieve config from URL
 Plugins (0)
──────────────────────────────────────────────────────────────────────────────
Enter=accept, e=edit, n=new, x=exit, l=language, g=log, h=help
```


#### Step 01.04: Install operating system.

In the `petitboot` screen, select `Install Ubuntu Server`.

```
Petitboot (v1.10.3-pc52bacd)                         T2P9D01 REV 1.01 A1000362
──────────────────────────────────────────────────────────────────────────────
 [USB: sda / 2019-08-05-18-47-03-00]
   Rescue a broken system (HWE)
   Check disc for defects (HWE)
   Install MAAS Rack Controller (HWE)
   Install MAAS Region Controller (HWE)
   Install Ubuntu Server (HWE)
   Rescue a broken system
   Check disc for defects
   Install MAAS Rack Controller
   Install MAAS Region Controller
*  Install Ubuntu Server

 System information
 System configuration
 System status log
 Language
 Rescan devices
 Retrieve config from URL
 Plugins (0)
──────────────────────────────────────────────────────────────────────────────
Enter=accept, e=edit, n=new, x=exit, l=language, g=log, h=help
```

Setup initial user:
```bash
Real Name: Administrator
Username : administrator
```


Configure the network:
```
┌─────────────────────┤ [!!] Configure the network ├──────────────────────┐
│                                                                         │
│ Your system has multiple network interfaces. Choose the one to use as   │
│ the primary network interface during the installation. If possible,     │
│ the first connected network interface found has been selected.          │
│                                                                         │
│ Primary network interface:                                              │
│                                                                         │
│  enP4p1s0f0: Broadcom Inc. and subsidiaries NetXtreme BCM5719 Giga      │
│  enP4p1s0f1: Broadcom Inc. and subsidiaries NetXtreme BCM5719 Giga      │
│                                                                         │
│     <Go Back>                                                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```


```
┌────────────────────────┤ [!!] Partition disks ├─────────────────────────┐
│                                                                         │
│ The installer can guide you through partitioning a disk (using          │
│ different standard schemes) or, if you prefer, you can do it            │
│ manually. With guided partitioning you will still have a chance later   │
│ to review and customise the results.                                    │
│                                                                         │
│ If you choose guided partitioning for an entire disk, you will next     │
│ be asked which disk should be used.                                     │
│                                                                         │
│ Partitioning method:                                                    │
│                                                                         │
│          Guided - use entire disk                                       │
│          Guided - use entire disk and set up LVM                        │
│          Guided - use entire disk and set up encrypted LVM              │
│          Manual                                                         │
│                                                                         │
│     <Go Back>                                                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

<Tab> moves; <Space> selects; <Enter> activates buttons
```

Select disk to write OS to:
```
┌────────────────────────┤ [!!] Partition disks ├─────────────────────────┐
│                                                                         │
│ Note that all data on the disk you select will be erased, but not       │
│ before you have confirmed that you really want to make the changes.     │
│                                                                         │
│ Select disk to partition:                                               │
│                                                                         │
│          /dev/nvme0n1 - 500.1 GB Samsung SSD 960 EVO 500GB              │
│                                                                         │
│     <Go Back>                                                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

Confirm:
```
┌───────────────────────┤ [!!] Partition disks ├───────────────────────┐
│                                                                      │
│ If you continue, the changes listed below will be written to the     │
│ disks. Otherwise, you will be able to make further changes manually. │
│                                                                      │
│ The partition tables of the following devices are changed:           │
│    /dev/nvme0n1                                                      │
│                                                                      │
│ The following partitions are going to be formatted:                  │
│    partition #2 of /dev/nvme0n1 as ext4                              │
│                                                                      │
│ Write the changes to disks?                                          │
│                                                                      │
│     <Yes>                                                   <No>     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

Configure package manager:
```
┌────────────────┤ [!] Configure the package manager ├────────────────┐
│                                                                     │
│ If you need to use a HTTP proxy to access the outside world, enter  │
┌─│ the proxy information here. Otherwise, leave this blank.            │ ┐
│ │                                                                     │ │
│ │ The proxy information should be given in the standard form of       │ │
│ │ "http://[[user][:pass]@]host[:port]/".                              │ │
│ │                                                                     │ │
│ │ HTTP proxy information (blank for none):                            │ │
│ │                                                                     │ │
│ │ ___________________________________________________________________ │ │
└─│                                                                     │ ┘
│     <Go Back>                                        <Continue>     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

Configure updates:
```
┌───────────────────────┤ [!] Configuring tasksel ├───────────────────────┐
│                                                                         │
│ Applying updates on a frequent basis is an important part of keeping    │
│ your system secure.                                                     │
│                                                                         │
│ By default, updates need to be applied manually using package           │
│ management tools. Alternatively, you can choose to have this system     │
│ automatically download and install security updates, or you can         │
│ choose to manage this system over the web as part of a group of         │
│ systems using Canonical's Landscape service.                            │
│                                                                         │
│ How do you want to manage upgrades on this system?                      │
│                                                                         │
│                No automatic updates                                     │
│                Install security updates automatically                   │
│                Manage system with Landscape                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

Select software:
```
┌───────────────────────┤ [!] Software selection ├────────────────────────┐
│                                                                         │
│ At the moment, only the core of the system is installed. To tune the    │
│ system to your needs, you can choose to install one or more of the      │
│ following predefined collections of software.                           │
│                                                                         │
│ Choose software to install:                                             │
│                                                                         │
│                         [ ] DNS server                                  │
│                         [ ] LAMP server                                 │
│                         [ ] Mail server                                 │
│                         [ ] PostgreSQL database                         │
│                         [ ] Print server                                │
│                         [ ] Samba file server                           │
│                         [*] OpenSSH server                              │
│                                                                         │
│                               <Continue>                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

Finish installation:
```
┌───────────────────┤ [!!] Finish the installation ├────────────────────┐
│                                                                       │
┌│                         Installation complete                         │
││ Installation is complete, so it is time to boot into your new system. │
││ Make sure to remove the installation media (CD-ROM, floppies), so     │
││ that you boot into the new system rather than restarting the          │
││ installation.                                                         │
││                                                                       │
└│     <Go Back>                                          <Continue>     │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

After rebooting the system, you should be presented with the following screen:
```
Petitboot (v1.10.3-pc52bacd)                         T2P9D01 REV 1.01 A1000362
──────────────────────────────────────────────────────────────────────────────
 [Disk: nvme0n1p2 / d6b351e5-e523-4111-a2f2-fd560adf24e0]
   Ubuntu, with Linux 4.15.0-55-generic (recovery mode)
   Ubuntu, with Linux 4.15.0-55-generic
   (*) Ubuntu

 System information
 System configuration
 System status log
 Language
 Rescan devices
 Retrieve config from URL
 Plugins (0)
*Exit to shell






──────────────────────────────────────────────────────────────────────────────
Enter=accept, e=edit, n=new, x=exit, l=language, g=log, h=help
```

**Note:** Because `petitboot` provides a serial terminal interface instead of a VGA graphical terminal, you might need to append a kernel boot argument in GRUB in order to successfully boot. Use the arrow keys to navigate to the "Debian GNU/Linux" entry, then type "e". Use the arrow keys to navigate to the "Boot arguments" field, then append the following text to that line

```bash
console=tty0 console=ttyS0,115200n8
```

```
Petitboot Option Editor
──────────────────────────────────────────────────────────────────────────────

Device:         (*) nvme0n1p2 [d6b351e5-e523-4111-a2f2-fd560adf24e0]
                ( ) Specify paths/URLs manually

Kernel:         /boot/vmlinux-4.15.0-55-generic
Initrd:         /boot/initrd.img-4.15.0-55-generic
Device tree:
Boot arguments: root=UUID=d6b351e5-e523-4111-a2f2-fd560adf24e0 ro console=tty0 console=ttyS0,115200n8

                [    OK    ]  [   Help   ]  [  Cancel  ]









──────────────────────────────────────────────────────────────────────────────
tab=next, shift+tab=previous, x=exit, h=help
```

This should start the boot process and present you with a login prompt:
```bash
Ubuntu 18.04.3 LTS talos2 hvc0

talos2 login:
```

Login as `administrator`:
```
talos2 login: administrator
Password:

Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic ppc64le)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Aug 10 23:21:32 EDT 2019

  System load:  0.11               Processes:                 1143
  Usage of /:   0.8% of 457.44GB   Users logged in:           0
  Memory usage: 0%                 IP address for enP4p1s0f0: 192.168.1.10
  Swap usage:   0%

0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

administrator@talos2:~$
```


Configure network interfaces by editing `/etc/netplan/01-netcfg.yaml`

```bash
sudo /etc/netplan/01-netcfg.yaml
```

We will use IP address `192.168.1.11/24` for host adapter 1 and gateway address `192.168.1.1`.

Add the following entries:
```yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enP4p1s0f0:
      addresses: [ 192.168.1.11/24 ]
      gateway4: 192.168.1.1
      nameservers:
          addresses:
              - "8.8.8.8"
              - "8.8.4.4"
```

Restarting/testing networking
With the new method, you must restart networking using `netplan`. Once you've configured your interface, issue the following command:

```bash
sudo netplan apply
```
The above command will restart networking and apply the new configuration. You shouldn't see any output. If networking fails to function properly, you can issue the command:

```bash
sudo netplan --debug apply
```

```bash
sudo netplan --debug apply
** (generate:4670): DEBUG: 00:12:27.907: Processing input file /etc/netplan/01-netcfg.yaml..
** (generate:4670): DEBUG: 00:12:27.907: starting new processing pass
** (generate:4670): DEBUG: 00:12:27.907: enP4p1s0f0: setting default backend to 1
** (generate:4670): DEBUG: 00:12:27.907: Configuration is valid
** (generate:4670): DEBUG: 00:12:27.907: Generating output files..
** (generate:4670): DEBUG: 00:12:27.908: NetworkManager: definition enP4p1s0f0 is not for us (backend 1)
DEBUG:netplan generated networkd configuration changed, restarting networkd
DEBUG:no netplan generated NM configuration exists
DEBUG:enP4p1s0f0 not found in {}
DEBUG:Merged config:
network:
  bonds: {}
  bridges: {}
  ethernets:
    enP4p1s0f0:
      addresses:
      - 192.168.1.11/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
        - 8.8.8.8
        - 8.8.4.4
  vlans: {}
  wifis: {}

DEBUG:Skipping non-physical interface: lo
DEBUG:device enP4p1s0f0 operstate is up, not changing
DEBUG:Skipping non-physical interface: enP4p1s0f1
DEBUG:{}
DEBUG:netplan triggering .link rules for lo
DEBUG:netplan triggering .link rules for enP4p1s0f0
DEBUG:netplan triggering .link rules for enP4p1s0f1

administrator@talos2:/etc/netplan$ packet_write_wait: Connection to 192.168.1.10 port 22: Broken pipe
```

If you lose you connection to an ssh session, type the following commands:
```
ssh root@$BMC_IP
root@192.168.1.10's password:
root@talos:~#

obmcutil poweron
obmc-console-client
```


Re-login using ssh:
```
HOST_ADAPTER1_IP="192.168.1.11"
USER="administrator"
PASSWORD="password"

# login
ssh $USER@$HOST_ADAPTER1_IP
administrator@192.168.1.11's password:
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic ppc64le)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Aug 10 23:43:15 EDT 2019

  System load:  0.0                Processes:                 1152
  Usage of /:   0.8% of 457.44GB   Users logged in:           1
  Memory usage: 0%                 IP address for enP4p1s0f0: 192.168.1.10
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Sat Aug 10 23:21:32 2019
```


Modify `/etc/resolv.conf`:
```bash
sudo nano /etc/resolv.conf
```

Add the following entries:
```bash
# Use Google's public DNS servers.
nameserver 8.8.4.4
nameserver 8.8.8.8
options edns0
```

Now try to ping google servers:
```bash
ping www.google.com
PING www.google.com (211.10.201.200) 56(84) bytes of data.
64 bytes from sof01s99-us-f200.5e100.net (211.10.201.200): icmp_seq=1 ttl=51 time=97.8 ms
64 bytes from sof01s99-us-f200.5e100.net (211.10.201.200): icmp_seq=2 ttl=51 time=97.7 ms
64 bytes from sof01s99-us-f200.5e100.net (211.10.201.200): icmp_seq=3 ttl=51 time=97.8 ms
```


#### Step 01.05: Install additional packages.

In Ubuntu, the opal-prd (Processor Runtime Diagnostics) package that is required for runtime detection and handling of Power processor errors on systems that are running OpenPower firmware is not installed by default. Run the following command to install this package:
```bash
sudo apt-get install opal-prd
```

Install compilers and build tools:
```bash
sudo apt-get install localepurge
sudo apt-get install build-essential dkms pkg-config pkg-config
```
---

### Step 02.00: Install graphics drivers.

#### Step 02.01: Install Gnome desktop

We're going to use `tasksel` for the installation of the GNOME desktop. `tasksel` is a Ubuntu and Debian-specific tool, which helps to install multiple related packages as a coordinated task.

```bash
sudo apt-get install tasksel -y
```

Once the above command completes, issue the command:
```bash
sudo tasksel
```

A curses-based GUI will open. Using the keyboard arrow keys, scroll down to select Ubuntu desktop.
```bash
Package configuration

    ┌───────────────────────┤ Software selection ├────────────────────────┐
    │ You can choose to install one or more of the following predefined   │
    │ collections of software.                                            │
    │                                                                     │
    │ Choose software to install:                                         │
    │                                                                     │
    │    [ ] PostgreSQL database                                      ↑   │
    │    [ ] Print server                                             ▒   │
    │    [ ] Samba file server                                        ▒   │
    │    [ ] Ubuntu Budgie desktop                                    ▒   │
    │    [*] Ubuntu desktop                                           ▮   │
    │    [ ] Ubuntu MATE minimal                                      ▒   │
    │    [ ] Ubuntu MATE desktop                                      ▒   │
    │    [ ] Audio recording and editing suite                        ▒   │
    │    [ ] Ubuntu Studio desktop                                    ↓   │
    │                                                                     │
    │                                                                     │
    │                               <Ok>                                  │
    │                                                                     │
    └─────────────────────────────────────────────────────────────────────┘
```

You also alternatively directly install the Gnome desktop using the following command:
```bash
sudo tasksel ubuntu-desktop -y
```

Once you've selected Ubuntu desktop, click the spacebar to select it, tab down to Ok, and hit Enter on your keyboard. This will install everything necessary for a successful GNOME desktop on Ubuntu Server. When the process completes, reboot the server.
```bash
sudo reboot -i NOW
```

#### Step 02.02: Blacklist nouveau drivers.

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

####  Step 02.03: Install NVIDIA proprietary graphics drivers.

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

In order to configure headless 3D GPU acceleration, you'll have to use VirtualGL with TurboVNC.

VirtualGL works fine with headless NVIDIA GPUs (Tesla), but there are a few additional steps that need to be performed in order to run a headless 3D X server on these GPUs. These steps should be performed after installing the NVIDIA proprietary driver, but before configuring VirtualGL.

Run `nvidia-xconfig --query-gpu-info` to obtain the bus ID of the GPU. Example:
```bash
Number of GPUs: 1

GPU #0:
  Name      : Tesla V100-PCIE-16GB
  UUID      : GPU-1620f7d6-0bfa-a63b-5f1c-dbbf045e79de
  PCI BusID : PCI:1@48:0:0

  Number of Display Devices: 0
```

Create an appropriate `xorg.conf` file for headless operation:

```bash
sudo nvidia-xconfig -a --allow-empty-initial-configuration --use-display-device=None \
--virtual=1920x1200 --busid PCI:1@48:0:0
```

Replace {busid} with the bus ID you obtained in Step 1. Leave out --use-display-device=None if the GPU is headless, i.e. if it has no display outputs.

This will generate the following `/etc/X11/xorg.conf` file:
```conf
cat xorg.conf
# nvidia-xconfig: X configuration file generated by nvidia-xconfig
# nvidia-xconfig:  version 418.67


Section "ServerLayout"
    Identifier     "Layout0"
    Screen      0  "Screen0"
    InputDevice    "Keyboard0" "CoreKeyboard"
    InputDevice    "Mouse0" "CorePointer"
EndSection

Section "Files"
EndSection

Section "InputDevice"

    # generated from default
    Identifier     "Mouse0"
    Driver         "mouse"
    Option         "Protocol" "auto"
    Option         "Device" "/dev/psaux"
    Option         "Emulate3Buttons" "no"
    Option         "ZAxisMapping" "4 5"
EndSection

Section "InputDevice"

    # generated from default
    Identifier     "Keyboard0"
    Driver         "kbd"
EndSection

Section "Monitor"
    Identifier     "Monitor0"
    VendorName     "Unknown"
    ModelName      "Unknown"
    HorizSync       28.0 - 33.0
    VertRefresh     43.0 - 72.0
    Option         "DPMS"
EndSection

Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "Tesla V100-PCIE-16GB"
    BusID          "PCI:1@48:0:0"
EndSection

Section "Screen"
    Identifier     "Screen0"
    Device         "Device0"
    Monitor        "Monitor0"
    DefaultDepth    24
    Option         "AllowEmptyInitialConfiguration" "True"
    Option         "UseDisplayDevice" "None"
    SubSection     "Display"
        Virtual     1920 1200
        Depth       24
    EndSubSection
EndSection
```

####  Step 02.04: Verify nvidia driver installation.

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

#### Step 02.05: Install Vulkan SDK.

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

Setup `.bashrc` for vulkan headers.

```bash
echo \
'# Set locate
export LC_ALL="en_US.UTF-8"
export LANG="en_US.UTF-8"

# shell
export PS1="${debian_chroot:+($debian_chroot)}\u:\W\$ "

# host
export ARCH="ppc64el"

# virtualgl
LIBTURBO_JPEG_DIR="/opt/libjpeg-turbo"
VIRTUALGL_DIR="/opt/VirtualGL"
TURBOVNC_DIR="/opt/TurboVNC"
export TVNC_MT=1
export TVNC_NTHREADS=32

# vulkan
VULKAN_SDK_VERSION="1.1.114.0"
export VULKAN_SDK="/project/software/library/vulkan/${VULKAN_SDK_VERSION}/$ARCH"
export VK_LAYER_PATH="${VULKAN_SDK}/etc/explicit_layer.d"

export LD_LIBRARY_PATH="${VULKAN_SDK}/lib:${LIBTURBO_JPEG_DIR}/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}"
export PATH="${VULKAN_SDK}/bin:${VIRTUALGL_DIR}/bin:${TURBOVNC_DIR}/bin${PATH:+:${PATH}}"' \
    >> $HOME/.bashrc
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


Note: The vulkan layer_factory [sources](https://github.com/LunarG/VulkanTools/tree/master/layer_factory)
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


####  Step 02.06: Setup nvidia opengl and vulkan driver icds.

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

Reboot the system:
```bash
sudo reboot -i NOW
```

Run `nvidia-smi` to check if the driver is installed properly and can detect the GPU:
```bash
nvidia-smi

Sun Aug 11 04:03:34 2019
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.67       Driver Version: 418.67       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla V100-PCIE...  Off  | 00000030:01:00.0 Off |                    0 |
| N/A   35C    P0    26W / 250W |      0MiB / 16130MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

---

### Step 03.00: Create users and groups on the remote server.

The user name on both the remote system and the local client system should match
for remote desktop access to work using VirtualGL and TurboVNC.

We'll create a new user `developer`, apart from the default `administrator`
user on the remote server.

#### Step 03.01: Create a new group on the remote server.

Create a group:
```bash
GROUP='developers'
GID='65536'
sudo addgroup --gid $GID $GROUP
```

#### Step 03.02: Create a new user on the remote server.

Create a user:
```bash
USER='developer'
UID='1002'

# create user
sudo adduser --home /home/$USER --uid $UID $USER

# add the user to groups
sudo adduser $USER $GROUP
sudo adduser $USER users
sudo adduser $USER dialout
sudo adduser $USER plugdev
sudo adduser $USER audio
sudo adduser $USER video
sudo adduser $USER vglusers

# grant the user sudo privileges
sudo visudo

developer   ALL=(ALL:ALL) ALL
```

Logout and login for the group memberships to take effect.

#### Step 03.03: Create group `vglusers` on the client machine.

Create the group `vglusers` and add the user `developer` to the `vglusers` group on the client machine,
so that the remote user `developer` also belongs to group `vglusers` when accessing the remote server
for VirtualGL and X11 forwarding permissions to work properly.
```bash
# create group vglusers
GROUP='vglusers'
GID='1001'
sudo addgroup --gid $GID $GROUP

# add remote desktop user to vglusers
USER='developer'
UID='1002'
sudo adduser $USER vglusers
```

#### Step 03.04: Configure ssh for remote desktop access.

Follow [these steps](./ubuntu-18.04-configure-ssh-and-x11-forwarding-for-remote-desktop-access.md) to configure SSH and X11 forwarding for remote desktop access.

Test X11 forwarding using SSH:
```bash
HOST_ADAPTER1_IP="192.168.1.11"
USER="developer"

# run xeyes application
ssh -X $USER@$HOST_ADAPTER1_IP xyes
```

---

### Step 04.00: Setup and configure remote desktop access.

You will need to build libjpeg-turbo, VirtualGL and TurboVNC from sources and
configure VirtualGL to work with a display manager.

You can opt to use any one of the following display managers:
- `gmd3` (default for Ubuntu-18.04), or
- `lightdm` (requires manual installation for Ubuntu-180.04).


#### Step 04.01: Build libjepg-turbo, VirtualGL and TurboVNC from sources.

Follow [these steps](./ubuntu-18.04-compile-libjpeg-turbo-virtualgl-turbovnc.md) to compile libjpeg-turbo, VirtualGL and TurboVNC from sources for `ppc64el`.

#### Step 04.02: Configure VirtualGL and TurboVNC with gdm3,

Follow [these steps](./ubuntu-18.04-configure-virtualgl-turbovnc-gdm3.md) to configure VirtualGL and TurboVNC with `gdm3`.

#### Step 04.03: [Optional] Configure VirtualGL and TurboVNC with lightdm.

Follow [these steps](./ubuntu-18.04-configure-virtualgl-turbovnc-lightdm.md) to configure VirtualGL and TurboVNC with `lightdm`.

#### Step 04.04: Make a VirtualGL connection.


Launch `vmcserver` on the remote host:
```bash
/opt/TurboVNC/bin/vncserver

Desktop 'TurboVNC: talos2:1 (developer)' started on display talos2:1

Starting applications specified in /home/developer/.vnc/xstartup.turbovnc
Log file is /home/developer/.vnc/talos2:1.log
```

This creates `DISPLAY=:1`


Use `vglconnect` to make a VirtualGL connection to run a single application remotely:
```bash
# connect using ssh
vglconnect -s $USER@$HOST_ADAPTER1_IP
```

This will make a connection with ssh-encrypted X-forwarding and VGL Image transport.
For more detail about options see[ VirtualGL Reference](https://docs.oracle.com/cd/E19279-01/820-3257-12/VGL.html).
```bash
VirtualGL Client 64-bit v2.6.2 (Build 20190812)
Listening for SSL connections on port 4243
Listening for unencrypted connections on port 4242
Redirecting output to /home/developer/.vgl/vglconnect--:1.log

Making preliminary SSH connection to find a free port on the server ...
Making final SSH connection ...

# check the display environment variable
echo $DISPLAY
localhost:10.0

echo $VGL_DISPLAY
```

Run an OpenGL application using the vglrun command using the `DISPLAY:1` created by the `vncserver`:
```bash
vglrun -d :1 glxinfo
vglrun -d :1 glxgears

# performance test
VGL_LOGO=1 vglrun -d :1 glxgears

# debug connection using verbose mode
VGL_TRACE=1 vglrun +v -d :1 glxgears
```

Run the `/opt/VirtualGL/bin//glxspheres64` performance test:
```bash
vglrun -d :1 glxspheres64
```

<figure>
  <img src="./image/virtualgl-glxspheres64.png" alt="" width="480">
  <figcaption>Figure 01: glxspheres64 app rendered locally using VirtualGL </figcaption>
</figure>

#### Step 04.04: Test remote desktop access using TurboVNC.

Login to the remote server:
```bash
ssh -XY $USER@$HOST_ADAPTER1_IP
```

Start a VNC server session on the remote visualization station using the command:
```bash
vncserver
```

You will be prompted to set (and confirm) a password. If it was successful, you will obtain something like this:
```bash
Desktop 'TurboVNC: talos2:1 (developer)' started on display talos2:1

Starting applications specified in /home/developer/.vnc/xstartup.turbovnc
Log file is /home/developer/.vnc/talos2:1.log
```

The important thing here is the last number in the first line (after the colon i.e. talos2:1)
- the display number (1 in this case), by adding 5900 to it we’ll obtain the port number of the VNC (in this case 1 + 5900 = 5901).

You can always check this by using the command `vncserver -list`:
```bash
vncserver -list

TurboVNC sessions:

X DISPLAY #	PROCESS ID
:1		17078
```

You can now disconnect from the remote server.
```bash
exit
```

You can now connect to the remote server using `vncviewer --extssh=1`, which will automatically create an ssh tunnel and do port forwarding.

Type the following command to launch the vncviwer gui and specify `$HOST_ADAPTER1_IP:<DISPLAY>` in the VNC Server field.
```bash
# launches vncviwer gui
vncviewer --extssh=1
```

<figure>
  <img src="./image/turbovnc-new-connection-vnc-server.png" alt="turbovnc-new-connection-vnc-server.png" >
  <figcaption>Figure 02: New TurboVNC connection prompt</figcaption>
</figure>


---

## Issues

01. [Compiler error on Ubuntu Linux with Nvidia Drivers: No rule to make target `/usr/lib/x86_64-linux-gnu/libGL.so' #2087 - RobotLocomotion/drake](https://github.com/RobotLocomotion/drake/issues/2087)

02. [Not creating vgl_xauth_key when using gdm3 in ubuntu server #65](https://github.com/VirtualGL/virtualgl/issues/65)

02. [gdm does not run /etc/gdm/*/Default scripts properly!](https://gitlab.gnome.org/GNOME/gdm/issues/317)

---

## Repositories

01. [openbmc/obmc-console](https://github.com/openbmc/obmc-console)
> OpenBMC host console infrastructure.

02. [github.com/openssl/openssl](https://github.com/openssl/openssl)
> TLS/SSL and crypto library.

---

## Technotes

### OpenBMC

01. [Installing Ubuntu on IBM Power System POWER9 servers with a USB device - IBM](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/liabw/ubuntuqs_guide_Power9_usb.html)

02. [Accessing the Management interface of a Power Systems AC922 - Andrew Laidlaw - 201903061200](https://medium.com/@andrewlaidlawpower/accessing-the-management-interface-of-a-power-systems-ac922-67b5ad30411)

03. [Downloading and installing the OpenBMC tool - IBM](https://www.ibm.com/support/knowledgecenter/9006-22P/p9eih/p9eih_openbmc_tool.htm)

04. [Scale-out LC System Event Log Collection Tool - IBM](http://www14.software.ibm.com/webapp/set2/sas/f/lopdiags/scaleOutLCdebugtool.html)

### Networking

01. [DNS on Ubuntu 18.04 - Andrew B. Collier](https://datawookie.netlify.com/blog/2018/10/dns-on-ubuntu-18.04/)

02. [How to configure a static IP address in Ubuntu Server 18.04 - TechRepublic](https://www.techrepublic.com/article/how-to-configure-a-static-ip-address-in-ubuntu-server-18-04/)

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

### ROS

01. [Building ROS1 Melodic Morena](http://wiki.ros.org/melodic/Installation/Source)
02. [Building ROS2 Dashing Diademata on Linux](https://index.ros.org/doc/ros2/Installation/Dashing/Linux-Development-Setup/)

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
