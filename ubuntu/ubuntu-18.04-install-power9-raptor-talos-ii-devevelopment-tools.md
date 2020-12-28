# Ubuntu-18.04 Installation Guide for IBM Power9 - Raptor Talos II Secure Workstation - Install Development Tools

## Overview

This document describes the setup procedure for installing common development tools and libraries on the Talos II secure workstation.

## Procedure

### Step 01.00: Install applications

#### Step 01.01: Install google chrome.

```bash
# update sources.list
sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list'

# add signing key
wget -qO - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

# verify
apt-key list

pub   dsa1024 2007-03-08 [SC]
      4CCA 1EAF 950C EE4A B839  76DC A040 830F 7FAC 5991
uid           [ unknown] Google, Inc. Linux Package Signing Key <linux-packages-keymaster@google.com>
sub   elg2048 2007-03-08 [E]

# install
sudo apt update
sudo apt install google-chrome-stable
```

#### Step 01.02: Install atom editor.

```bash
# install
sudo add-apt-repository ppa:webupd8team/atom
sudo apt update; sudo apt install atom

# remove
sudo apt remove --purge atom
```


### Step 03.00: Setup and configure docker.

#### Step 03.01: Remove an existing docker installation.

Stop the docker service:
```bash
sudo systemctl stop docker

sudo systemctl status docker
```

Remove docker:
```bash
sudo apt remove docker docker-ce docker-compose
```

Remove docker configuration files

If docker has been run once without user namespace remapping, `/var/lib/docker`
will contain configuration files, images and containers.

We should ideally delete these files and images, to save on space before
enabling user namespace remapping.

**Optional:** If you have previously installed Ubuntu on a `btrfs` filesystem
you will have to delete existing `btrfs` subvoluments. If not, you can skip this step.

Delete `btrfs` subvolumes if it exists.
```bash
sudo -s
cd /var/lib/docker
btrfs subvolume delete btrfs/subvolumes/*

# delete subvolumes for a specific user
cd /var/lib/docker/1026.999
btrfs subvolume delete btrfs/subvolumes/*
```

Remove existing docker configuration files:
```bash
sudo -s
rm -Rf /var/lib/docker
```

#### Step 03.02: Configure user-namespace remapping.

Before installing docker, we will add some configuration files to setup
usernamespace remapping.

Specify subuid and subgid.

Edit `/etc/subuid`
```bash
sudo nano /etc/subuid

<username>:1026:1
<username>:100000:65536
```

Get the docker group id:
```bash
getent group docker
docker:x:999:<username>
```

Use the obtained gid `999` to set the subgid, for specifying the docker group
for setting the user group, for user namespace remapping. This will allow files
created on the host to belong to $USER:docker.

Edit /etc/subgid
```bash
sudo nano /etc/subgid

<username>:999:1
<username>:100000:65536
```

Edit /etc/docker/daemon.json
```bash
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json
```

```json
{
    "dns": ["8.8.8.8", "8.8.4.4"],
    "userns-remap": "<username>",
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

**Note:** Ensure that you don't have extraneous commas in the daemon.json file.
It will cause errors and prevent the docker daemon from starting.


#### Step 03.03: Install Docker CE.

```bash
sudo apt-get update

# install required packages
sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# add docker repository
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# update repositories
sudo apt update

# make sure you are about to install from the Docker repo instead of the default Ubuntu repo
apt-cache policy docker-ce

# install
sudo apt install docker-ce

# check status
sudo systemctl status docker

# add user to the docker group
sudo usermod -aG docker <username>
```

Configure Docker to start on boot:
```bash
sudo systemctl enable docker
```

Reboot the system. After the system comes up, check that the kernel options were properly assigned and that the docker service is running with user namespaces enabled.

This can be verified by looking at the contents of `/var/lib/docker` and verifying the existence of new user-specific folders:
```bash
sudo -s
cd /var/lib/docker
ls -la

total 12
drwx--x--x  3 root  root   4096 Nov  8 10:31 .
drwxr-xr-x 62 root  root   4096 Nov  8 10:31 ..
drwx------ 14 <username> docker 4096 Nov  8 10:31 1026.999
```

If you need to restart the docker daemon and docker service, enter the following command:

```bash
sudo systemctl daemon-reload; sudo systemctl restart docker;
```
or

```bash
sudo dockerd --userns-remap="<username>:<username>"
```
if you didn't make an entry in daemon.json

The Docker Engine applies the same user namespace remapping rules to all containers, regardless of who runs a container or who executes a command within a container.

#### Step 03.04: Install nvidia-docker2.

Configure repositories:
```
# add nvidia-docker repository
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
```

Install the `nvidia-docker2` package and reload the docker daemon configuration:
```
sudo apt-get install nvidia-docker2
sudo pkill -SIGHUP dockerd
```

#### Step 03.05: Install docker-compose.

Run this command to download the latest version of Docker Compose:
```
# download
sudo curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

# apply executable persmissions
sudo chmod +x /usr/local/bin/docker-compose

# test the installation
docker-compose --version
docker-compose version 1.24.0, build 0aa59064
```

#### Step 03:06: Test running a docker image

Test the docker installation:
```bash
docker run --rm hello-world
docker run --rm -it ubuntu:xenial cat /etc/issue
Ubuntu 16.04.5 LTS \n \l
```

#### Step 03.07: Test NVIDIA Docker2 Image

Test NVIDIA OpenGL by running the glxgears example:
```
git clone https://gitlab.com/nvidia/samples.git /project/software/infrastructure/docker/nvidia/samples
cd /project/software/infrastructure/docker/nvidia/samples/opengl/ubuntu16.04/glxgears

# build the docker image
docker build \
  --file Dockerfile \
  -t nvidia/opengl:glxgears-ubuntu16.04 .

# run the docker image
xhost +local:root
docker run -it \
  --runtime=nvidia \
  -e DISPLAY \
  -e QT_GRAPHICSSYSTEM=native \
  -e QT_X11_NO_MITSHM=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  nvidia/opengl:glxgears-ubuntu16.04
xhost -local:root
```

Test NVIDIA CUDA image and check if GPUs are reported correctly by nvidia-smi:

```bash
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```


#### Step 03:06:Enable volume sharing

Create a set of common folders that will act as volume mount points for a docker container.

If you plan on using an external volume mount or a separate nvme drive dedicated for storing
docker images, create a symbolic link to the volume mount point or nvme drive instead.

Find disk usage:
```bash
df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        28G  9.3G   17G  36% /

df -h /media/<username>/nvme
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p1  469G   18G  428G   4% /media/<username>/nvme
```

```bash
mkdir -p ~/mount/backup ~/mount/data ~/mount/project ~/mount/tool
```



Alternatively, you can create a symbolic link to folders within the internal storage drive to your home folder:
```bash
mkdir -p /mnt/storage/mount
ln -s /mnt/storage/mount ~/mount
```

```bash
mkdir -p /mnt/storage/mount/backup /mnt/storage/mount/data /mnt/storage/mount/project /mnt/storage/mount/tool
```

If the docker user `developer` has `uid=1000` and `gid=1000`,
it will be user_namespace remapped to `uid=100999` and `gid=100999`.

When you launch a docker image and mount a volume, the volume mounts will initially
be owned by root within the container because of user namespace remapping. Change
the permission of one of the volume mounts from within the container, to double check
the resulting uid:gid on the host and then chown all the volume mount points from
the host.

```bash
# run this command from within the container
sudo chown 1000:1000 /project

# run this command from the host
cd ~/mount
ls -la
drwxrwxr-x. 1 100999 100999 464 Oct 29 19:46 project
```

Define an additional group on the host and add the current user to the group
to be able to share data generated by a non-root user within the docker container.

```bash
sudo addgroup --gid 100999 docker-developer
sudo usermod -aG docker-developer <username>
```

Change permissions recursively for the mount points so that they are accessible
to a non-root user from within the container:
```bash
sudo chmod -R ug+rw ~/mount; sudo chown -R 100999:docker-developer ~/mount;
```

If your `~/mount` folder is symbolically linked to `/mnt/storage/mount`, type the following commands to enable access from the docker container:
```bash
sudo chmod -R ug+w /mnt/storage/mount; sudo chown -R 100999:docker-developer /mnt/storage/mount;
```

Log-off and log back in for the folder permissions to take effect.



### Step 05.00: Configure NFS support.

#### Step 05.01: Install required packages for Ubuntu 18.04.

NFS support files common to client and server are available in the nfs-common package.
```bash
sudo apt-get install nfs-common
```

#### Step 05.02: Mount an NFS filesystem in read/write mode from Ubuntu 18.04.

Create nfs mount points:
```bash
mkdir ~/Repository
sudo mkdir -p            /backup /data /project /tool /media/<username>/diskstation/archive /media/<username>/diskstation/backup /media/<username>/diskstation/download
sudo chown -R <username> /backup /data /project /tool /media/<username>/diskstation/archive /media/<username>/diskstation/backup /media/<username>/diskstation/download
sudo chgrp -R developers /backup /data /project /tool /media/<username>/diskstation/archive /media/<username>/diskstation/backup /media/<username>/diskstation/download
```

To manually mount the nfs share:
```bash
sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/project /project
```

To automatically mount the NFS share, edit the /etc/fstab file as follows:

```bash
su
gedit /etc/fstab

# automount synology shared folders
<nas-device-ip-address>:/volume1/archive      /media/<username>/diskstation/archive  nfs auto 0 0
<nas-device-ip-address>:/volume1/backup       /media/<username>/diskstation/backup   nfs auto 0 0
<nas-device-ip-address>:/volume1/download     /media/<username>/diskstation/download nfs auto 0 0
<nas-device-ip-address>:/volume1/data         /data        nfs auto 0 0
<nas-device-ip-address>:/volume1/project      /project     nfs auto 0 0
<nas-device-ip-address>:/volume1/tool/ubuntu  /tool        nfs auto 0 0
```

Type the following command into a terminal to mount the shared folder
```bash
sudo mount -a
```

#### Step 05.03: Create an .aliases.sh file to make it easier to mount the nfs shares.

```bash
gedit ~/.aliases.sh
```

```bash
#!/bin/bash

# Filename    : ~/.aliases.sh
# Description : Script to setup aliases for the current user.

# Function to mount diskstataion volumes.
function mount_diskstation {
  sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/repository  /home/<username>/Repository;
  sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/archive     /media/<username>/diskstation/archive;
  sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/backup      /media/<username>/diskstation/backup;
  sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/download    /media/<username>/diskstation/download;
  sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/project     /data;
  sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/project     /project;
  sudo mount -t nfs -o proto=tcp,port=2050 <nas-device-ip-address>:/volume1/tool/ubuntu /tool;
}

# Function to unmount diskstation volumes.
function umount_diskstation {
  sudo umount /home/<username>/Repository;
  sudo umount /media/<username>/diskstation/archive;
  sudo umount /media/<username>/diskstation/backup;
  sudo umount /media/<username>/diskstation/download;
  sudo umount /data;
  sudo umount /project;
  sudo umount /tool;
}
```


### Step 06.00: Install basic packages.

#### Step 06.01: Install git.

```bash
sudo apt-get install git git-gui
```

Setup git global settings:

```bash
$ gedit ~/.gitconfig
```

```
[user]
    email = username@domain.com
    name = username
[color]
    ui = auto
    diff = auto
    status = auto
    branch = auto
[core]
    safecrlf = true
    whitespace = trailing-space,space-before-tab
[apply]
    whitespace = fix
[push]
    default = simple
[pull]
    rebase = true
[rebase]
    autoStash = true
```

#### Step 06.02: Install meld for performing visual diff.

```bash
sudo apt-get install meld
```

#### Step 06.03: Install cmake for generating cross platform build makefiles.

```bash
sudo apt-get install cmake
```

### Step 07.00: Common maintenance tasks.

#### Step 07.01: Clear recent history.

Click on Security & Privacy > Files & Applications
and clear usage data.

