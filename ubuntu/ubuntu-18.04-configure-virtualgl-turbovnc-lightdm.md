# Ubuntu-18.04 - Configure VirtualGL and TurboVNC with lightdm

This document describes how to configure VirtualGL and TurboVNC with lightdm desktop manager.

## Performance tweaks

In order to get better performance rendering remote workloads, you will need to:
- enable multi-threading,
- tweak encoding methods, and
- adjust remote desktop resolution.

### Multithreading

The TurboVNC Server can use multiple threads to perform image encoding and compression, thus allowing it to take advantage of multi-core or multi-processor systems. The server splits the screen vertically into N tiles, where N is the number of threads, and assigns each tile to a separate thread. The scalability of this algorithm is nearly linear when used with demanding 3D or video applications that fill most of the screen. However, whether or not multithreading improves the overall performance of TurboVNC depends largely on the performance of the viewer and the network. If either the viewer or the network is the primary performance bottleneck, then multithreading the server will not help. It will almost certainly have no effect on networks slower than 100 Megabit Ethernet or when using the Java TurboVNC Viewer as an applet.

To enable server-side multithreading, set the TVNC_MT environment variable to 1 on the server prior to starting vncserver. The default behavior is to use as many threads as there are cores on the server machine, but you can set the TVNC_NTHREADS environment variable to override this.

In order to get best performance over a remote connection, enable multi-threading and
specify the number of threads for TurboVNC to use for rendering before running `vncserver`:
```bash
export TVNC_MT=1
export TVNC_NTHREADS=32
/opt/TurboVNC/bin/vncserver
```

### TurboVNC viewer connection settings

Use the following settings for optimal performance over a remote network connection:
```yaml
Encoding:
  - Encoding method: Tight + Medium Quality JPEG

Connection:
  - Remote desktop size: 1440x900
```


### Maximizing performance of the Java TurboVNC viewer

**Accelerated JPEG Decoding**

The Java TurboVNC Viewer can be used as a standalone app, in which case it provides most of the same features as the native TurboVNC viewer. It can also load libjpeg-turbo through JNI to accelerate JPEG decoding, which gives the Java viewer similar performance to the native viewer in most cases. The TurboVNC Viewer on Mac and Linux/Un*x platforms is simply the Java TurboVNC Viewer packaged in such a way that it behaves like a native viewer. On Windows, the Java TurboVNC Viewer is packaged similarly, but it is included alongside the native TurboVNC Viewer, giving users a choice between the two. The Java TurboVNC Viewer packaging includes the libjpeg-turbo JNI library, which is automatically loaded when you launch the TurboVNC Viewer app on Mac, run the vncviewer script on Linux/Un*x, run the vncviewer-java.bat script on Windows, or launch “Java TurboVNC Viewer” from the Windows Start Menu. Thus, if you are running the Java TurboVNC Viewer in one of those ways, then no further action is needed to accelerate it.

If you suspect for whatever reason that JPEG decoding is not being accelerated, then the easiest way to check is to open the “Connection Info” dialog (after the connection to the server has been established) and verify that the “JPEG decompression” field says “Turbo”. If you are launching the Java TurboVNC Viewer from the command line, then it will also print a warning if it is unable to load libjpeg-turbo.

### Step 01.00: Configure remote desktop access.

#### Step 01.01: Configure ssh to enable X11 forwarding.

Edit `/etc/ssh/sshd_config` and ensure that X11 forwarding is enabled
and X11 doesn't use the local host:
:
```bash
X11Forwarding yes
X11UseLocalhost no
```

**Important:** If `X11UseLocalhost` is not set to `no`, you will get the following errors when connecting to the remote server from a remote client:
```bash
X11 connection rejected because of wrong authentication.
Error: Cant open display: localhost:10.0
```

Edit `/etc/ssh/ssh_config` and ensure that X11 forwarding is enabled:
```bash
Host *
ForwardX11 yes
```

#### Step 01.02: Configure virtualgl server with lightdm.

Ref:
- [6. Configuring a Linux or Unix Machine as a VirtualGL Server - VirtualGL](https://cdn.rawgit.com/VirtualGL/virtualgl/2.6.1/doc/index.html#hd006)
- [Not creating vgl_xauth_key when using gdm3 in ubuntu server #65](https://github.com/VirtualGL/virtualgl/issues/65)


Disable `gdm3` and install `lightdm`:
```bash
sudo systemctl disable gdm3
sudo apt install lightdm
sudo systemctl enable lightdm

# if enabling lightdm doesnt work with ubuntu-18.04, you can manually create the symlink
sudo ln -s /lib/systemd/system/lightdm.service /etc/systemd/system/display-manager.service

# you can also reconfigure lightdm to enable it
sudo dpkg-reconfigure lightdm
```

Check configuration:
```bash
lightdm --show-config

   [Seat:*]
A  allow-guest=false
C  greeter-wrapper=/usr/lib/lightdm/lightdm-greeter-session
D  guest-wrapper=/usr/lib/lightdm/lightdm-guest-session
E  user-session=ubuntu
F  greeter-session=unity-greeter
G  xserver-command=X -core

   [LightDM]
B  backup-logs=false

   [Seat:seat*]
H  greeter-setup-script=/opt/VirtualGL/bin/vglgenkey

Sources:
A  /usr/share/lightdm/lightdm.conf.d/50-disable-guest.conf
B  /usr/share/lightdm/lightdm.conf.d/50-disable-log-backup.conf
C  /usr/share/lightdm/lightdm.conf.d/50-greeter-wrapper.conf
D  /usr/share/lightdm/lightdm.conf.d/50-guest-wrapper.conf
E  /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
F  /usr/share/lightdm/lightdm.conf.d/50-unity-greeter.conf
G  /usr/share/lightdm/lightdm.conf.d/50-xserver-command.conf
H  /etc/lightdm/lightdm.conf
```

Check the status of the `lightdm` service
```bash
sudo service lightdm status
● lightdm.service - Light Display Manager
   Loaded: loaded (/lib/systemd/system/lightdm.service; indirect; vendor preset: enabled)
   Active: active (exited) since Mon 2019-08-12 02:53:58 EDT; 4min 10s ago
     Docs: man:lightdm(1)
    Tasks: 0 (limit: 22118)
   CGroup: /system.slice/lightdm.service

Aug 12 02:53:58 talos2 systemd[1]: Starting LSB: Start lightdm...
Aug 12 02:53:58 talos2 systemd[1]: Started LSB: Start lightdm.
```

Login as `root` and configure the virtualgl server:
```bash
sudo -s

# stop lightdm
service lightdm stop
service lightdm status

# rmmod nvidia driver modules before configuring the vglserver
rmmod nvidia_drm
rmmod nvidia_modeset
rmmod nvidia

# configure
/opt/VirtualGL/bin/vglserver_config
```

You will be asked three questions during this configuration. You can arrange things how you like, but for the most secure configuration, you will want to reply in the affirmative to each question.

```bash
"Restrict 3D X server access to vglusers group?"  Yes.
"Restrict framebuffer device access to vglusers group?"  Yes.
"Disable XTEST extension?"  Yes.
```

Here is a sample screen output:
```bash
1) Configure server for use with VirtualGL
2) Unconfigure server for use with VirtualGL
X) Exit

# select option 1
Choose:
1

Restrict 3D X server access to vglusers group (recommended)?
[Y/n]
y

Restrict framebuffer device access to vglusers group (recommended)?
[Y/n]
y

Disable XTEST extension (recommended)?
[Y/n]
y

... Creating vglusers group ...
groupadd: group 'vglusers' already exists
Could not add vglusers group (probably because it already exists.)
... Granting read permission to /etc/opt/VirtualGL/ for vglusers group ...
... Creating /etc/modprobe.d/virtualgl.conf to set requested permissions for
    /dev/nvidia* ...
... Granting write permission to /dev/nvidia0 /dev/nvidiactl for vglusers group ...
... Granting write permission to /dev/dri/card0 for vglusers group ...
... Modifying /etc/X11/xorg.conf to enable DRI permissions
    for vglusers group ...
... Adding vglgenkey to /etc/gdm3/Init/Default script ...
... Adding greeter-setup-script=vglgenkey to /etc/lightdm/lightdm.conf ...
... Creating /usr/share/gdm/greeter/autostart/virtualgl.desktop ...

Done. You must restart the display manager for the changes to take effect.
```

If you didn't `rmmod nvidia` and other dependent drivers, you will get the
additional message in after running `vglserver_config`:
```bash
IMPORTANT NOTE: Your system uses modprobe.d to set device permissions. You
must execute rmmod nvidia with the display manager stopped in order for the
new device permission settings to become effective.
```


Add the current user to the `vglusers` group:
```bash
# check vglusers group
cat /etc/group | grep vglusers
vglusers:x:1001:

# add users to vglusers group
sudo usermod -G vglusers -a 'root'
sudo usermod -G vglusers -a 'administrator'
sudo usermod -G vglusers -a 'developer'

# logout
exit
```

Login and check group membership:
```bash
id

uid=1002(developer) gid=1002(developer) groups=1002(developer),20(dialout),29(audio),44(video),46(plugdev),100(users),1001(vglusers),65536(developers)
```

Modify the lightdm configuration file to run on the first X server
by changing `[Seat:seat*]` to `[Seat:seat0]`:
```bash
cat /etc/lightdm/lightdm.conf

[SeatDefaults]
[Seat:seat0]
greeter-setup-script=/opt/VirtualGL/bin/vglgenkey
```


Optional: Check gdm3 configuration files:
```bash
cd /usr/share/gdm/greeter/autostart/

cat virtualgl.desktop

[Desktop Entry]
Type=Application
Exec=/opt/VirtualGL/bin/vglgenkey
```

Reboot the system:
```bash
sudo reboot -i NOW
```

Create an `~/.Xauthority` file for a user by logging in to the remote server using `ssh -X`:
```bash
HOST_ADAPTER1_IP="192.168.1.11"
USER="administrator"
PASSWORD="password"

ssh -XY $USER@$HOST_ADAPTER1_IP

# it will complain that .Xauthority does not exist but will create one for you.
/usr/bin/xauth:  file /home/administrator/.Xauthority does not exist
```

Check status of `lightdm` service:
```bash
service service lightdm status

● lightdm.service - Light Display Manager
   Loaded: loaded (/lib/systemd/system/lightdm.service; indirect; vendor preset: enabled)
   Active: failed (Result: exit-code) since Mon 2019-08-12 03:06:49 EDT; 41s ago
     Docs: man:lightdm(1)
  Process: 4539 ExecStart=/usr/sbin/lightdm (code=exited, status=1/FAILURE)
  Process: 4530 ExecStartPre=/bin/sh -c [ "$(basename $(cat /etc/X11/default-display-manager 2>/dev/null))" = "lightdm" ] (code=exited, status=0/SUCCESS)
 Main PID: 4539 (code=exited, status=1/FAILURE)

Aug 12 03:06:49 talos2 systemd[1]: lightdm.service: Service hold-off time over, scheduling restart.
Aug 12 03:06:49 talos2 systemd[1]: lightdm.service: Scheduled restart job, restart counter is at 5.
Aug 12 03:06:49 talos2 systemd[1]: Stopped Light Display Manager.
Aug 12 03:06:49 talos2 systemd[1]: lightdm.service: Start request repeated too quickly.
Aug 12 03:06:49 talos2 systemd[1]: lightdm.service: Failed with result 'exit-code'.
Aug 12 03:06:49 talos2 systemd[1]: Failed to start Light Display Manager.

# if it faileed to start, then restart the lightdm service
sudo service lightdm start

# check the status
sudo service lightdm status
● lightdm.service - Light Display Manager
   Loaded: loaded (/lib/systemd/system/lightdm.service; indirect; vendor preset: enabled)
   Active: active (running) since Mon 2019-08-12 03:07:49 EDT; 849ms ago
     Docs: man:lightdm(1)
  Process: 4984 ExecStartPre=/bin/sh -c [ "$(basename $(cat /etc/X11/default-display-manager 2>/dev/null))" = "lightdm" ] (code=exited, status=0/SUCCESS)
 Main PID: 4995 (lightdm)
    Tasks: 6 (limit: 22118)
   CGroup: /system.slice/lightdm.service
           ├─4995 /usr/sbin/lightdm
           └─5006 /usr/lib/xorg/Xorg -core :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch

Aug 12 03:07:49 talos2 systemd[1]: Starting Light Display Manager...
Aug 12 03:07:49 talos2 systemd[1]: Started Light Display Manager.
```

Ensure that the `/etc/opt/VirtualGL/vgl_xauth_key` file is generated:
```bash
# check for file existence
file /etc/opt/VirtualGL/vgl_xauth_key
/etc/opt/VirtualGL/vgl_xauth_key: X11 Xauthority data

# if it doesn't exist, generate file /etc/opt/VirtualGL/vgl_xauth_key
sudo /opt/VirtualGL/bin/vglgenkey

# file vgl_xauth_key does not exist, but gets generated
xauth:  file /etc/opt/VirtualGL/vgl_xauth_key does not exist

# check
cd /etc/opt/VirtualGL/
ls -la

total 12
drwxr-x--- 2 gdm  vglusers 4096 Aug 11 18:21 .
drwxr-xr-x 4 root root     4096 Aug 11 17:01 ..
-rw-r--r-- 1 root root       52 Aug 11 18:21 vgl_xauth_key
```

On GNU/Linux systems running an X11 display server, the file `~/.Xauthority` stores authentication cookies or cryptographic keys used to authorize connection to the display. In most cases, the authentication mechanism is a symmetric cookie which is referred to as a `Magic Cookie`. The same cookie is used by the server as well as the client.

Each X11 authentication cookie is under the control of the individual system authenticated user. Since the authetication cookie is stored as a plain text security token, the permissions on the `~/.Xauthority` file should be `rw` for the owner only, `600` in octal format. However, the permissions on the authorization file are not enforced.

A user can list, export, create, or delete authentication cookies using the xauth program. The following command will create an authoratization cookie for `DISPLAY 32`.

```bash
xauth add localhost:32 - `mcookie`
```

Manual creation and manipulation of cookies is usually not needed when using X11 forwarding with ssh, because `ssh` starts an X11 proxy on the remote machine and automatically generates authorization cookies on the local display. However, for certain configurations the authorization cookie may need to be manually created and copied to the local machine.

This can be done in an `ssh` session and then use `scp` to copy the cookie.

ssh into remote machine:
```bash
ssh -XY $USER@$HOST_ADAPTER1_IP
```

To verify that the application server is ready to run VirtualGL, log out of the server, log back into the server using SSH, and execute the following commands in the SSH session:

Logout:
```bash
exit
```

Login using ssh:
```bash
ssh -X $USER@$HOST_ADAPTER1_IP
```

Perform sanity checks:
```bash
# sanity check for restricted access to 3D X Server
xauth merge /etc/opt/VirtualGL/vgl_xauth_key

# check display
echo $DISPLAY
localhost:10.0

echo $VGL_DISPLAY

# check if an authorization cookie is present for the current X11 display
xauth list

# if there's a $DISPLAY environment variable set but no corresponding
# authorization cookie for that display number, you can create one
xauth add $DISPLAY - `mcookie`

# and verify that there is now a cookie:
xauth list
```

Copy that cookie and merge it into the local machine:
```bash
user@remote> xauth nextract ~/xcookie $DISPLAY
user@remote> exit
user@local> scp $USER@$HOST_ADAPTER1_IP:~/xcookie ~/xcookie
user@local> xauth nmerge ~/xcookie
```

And then verify that the cookie has been installed:
```bash
user@local> xauth list

apollo/unix:  MIT-MAGIC-COOKIE-1  <magic-cookie>
talos2/unix:10  MIT-MAGIC-COOKIE-1  <magic-cookie>
```


Check display info:
```bash
# check display
echo $DISPLAY
echo $VGL_DISPLAY

# get display info
xdpyinfo -display :10.0

xdpyinfo -display :10.0
/opt/VirtualGL/bin/glxinfo -display :10.0 -c

# check that the graphic card is indeed used and not using mesa
```

Try opening an X11 application from a remote client session:
```bash
ssh -XY $USER@$HOST_ADAPTER1_IP

# now you can open up a graphic application from the remote machine,
# try "gedit" and all should go along smoothly.
```

#### Step 01.03: Configure turbovnc server.

Launch `/opt/TurboVNC/bin/vncserver` on the server.
Possible options:
-otp: enable One Time Password
-geometry x: specify size of the window

```bash
/opt/TurboVNC/bin/vncserver

You will require a password to access your desktops.

Password:
Verify:

Desktop 'TurboVNC: talos2:1 (developer)' started on display talos2:1

Creating default startup script /home/developer/.vnc/xstartup.turbovnc
Starting applications specified in /home/developer/.vnc/xstartup.turbovnc
Log file is /home/developer/.vnc/talos2:1.log
```

#### Step 01.04: Connect using TurboVNC client from x86.

Install required packages:
```bash
sudo apt install default-jre
```

Launch vncserver on the remote system:
```bash
HOST_ADAPTER1_IP="192.168.1.11"
USER="developer"
vglconnect -s $USER@$HOST_ADAPTER1_IP

# on server
/opt/TurboVNC/bin/vncserver

Desktop 'TurboVNC: talos2:1 (developer)' started on display talos2:1

Starting applications specified in /home/developer/.vnc/xstartup.turbovnc
Log file is /home/developer/.vnc/talos2:1.log

# use vglrun
DISPLAY=":1" /opt/VirtualGL/bin/vglrun -d :1 glxgears
2745 frames in 5.0 seconds = 548.921 FPS
2825 frames in 5.0 seconds = 564.941 FPS
```

### Step 01.05: Connect using TurboVNC.

Connect using TurboVNC client:
```bash
/opt/TurboVNC/bin/vncviewer
```

The screen will appear black but X is launched.

In a terminal on the server, export the DISPLAY variable to the value used by vncserver:
```bash
export DISPLAY=:1 # if your vncserver started in DISPLAY :1
```

Launch an X App (with vglrun to get 3D acceleration)
```bash
sudo apt install meta-utils vulkan-utils
vglrun glxgears
vglrun vulkan-smoketest
```


```bash
HOST_ADAPTER1_IP="192.168.1.11"
USER="developer"
vglconnect -s $USER@$HOST_ADAPTER1_IP
vglconnect    $USER@$HOST_ADAPTER1_IP

# run glxgears
/opt/VirtualGL/bin/vglrun glxgears

# or
DISPLAY=:10.0 VGL_LOGO=1 vglrun -d :10.0 glxgears
```

VGL transport with a direct X11 connection:
```bash
vglconnect -x $USER@$HOST_ADAPTER1_IP
```

Perform headless rendering:
```bash
# on server
/opt/TurboVNC/bin/vncserver  -alwaysshared

Desktop 'TurboVNC: talos2:1 (administrator)' started on display talos2:1

Starting applications specified in /home/administrator/.vnc/xstartup.turbovnc
Log file is /home/administrator/.vnc/talos2:1.log

# on another client
DISPLAY=":1" /opt/VirtualGL/bin/vglrun -d :1 glxgears
```

List running vncserver sessions:
```bash
vncserver -list

TurboVNC sessions:

X DISPLAY #	PROCESS ID
:1		5291
```

Kill a vncserver session:
```bash
vncserver -kill :1
```

#### Step 01.07: Login to remote desktop

```bash
vgllogin -s $USER@$HOST_ADAPTER1_IP
vglconnect -s $USER@$HOST_ADAPTER1_IP
vglconnect -x $USER@$HOST_ADAPTER1_IP
```

#### Step 01.08: View remote desktop

Launch vncserver:
```bash
# on remote server
/opt/TurboVNC/bin/vncserver -alwaysshared

Desktop 'TurboVNC: talos2:1 (developer)' started on display talos2:1

Starting applications specified in /home/developer/.vnc/xstartup.turbovnc
Log file is /home/developer/.vnc/talos2:1.log
```

The above message shows that `vncserver` started on display `:1`

Connect using vncviewer:
```bash
# on client
/opt/TurboVNC/bin/vncviewer
```

In the server address, use the format `<HOSTNAME>:<DISPLAY>` to connect the
the `DISPLAY` session created by `vncserver`
