# Ubuntu-18.04 - Configure VirtualGL and TurboVNC with gdm3

This document describes how to configure VirtualGL and TurboVNC with gdm3 desktop manager.

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

#### Step 01.02: Configure virtualgl server with gdm3.

Ref:
- [6. Configuring a Linux or Unix Machine as a VirtualGL Server - VirtualGL](https://cdn.rawgit.com/VirtualGL/virtualgl/2.6.1/doc/index.html#hd006)
- [Not creating vgl_xauth_key when using gdm3 in ubuntu server #65](https://github.com/VirtualGL/virtualgl/issues/65)


Stop `gdm3` service:
```bash
sudo service gdm3 status
sudo service gdm3 stop
sudo service gdm3 status
```

Login as `root` and configure the virtualgl server:
```bash
sudo -s

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
... Creating /etc/opt/VirtualGL/ ...
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
... Disabling Wayland in /etc/gdm3/custom.conf ...

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

Login and check group membership on the remote server:
```bash
id

uid=1002(developer) gid=1002(developer) groups=1002(developer),20(dialout),29(audio),44(video),46(plugdev),100(users),1001(vglusers),65536(developers)
```

Restart the display manager:
```bash
# restart gdm3 service
sudo service gdm3 restart

# check status
sudo service gdm3 status
● gdm.service - GNOME Display Manager
   Loaded: loaded (/lib/systemd/system/gdm.service; static; vendor preset: enabled)
   Active: active (running) since Mon 2019-08-12 04:00:01 EDT; 2s ago
  Process: 4823 ExecStartPre=/usr/share/gdm/generate-config (code=exited, status=0/SUCCESS)
 Main PID: 4834 (gdm3)
    Tasks: 9 (limit: 22118)
   CGroup: /system.slice/gdm.service
           ├─4834 /usr/sbin/gdm3
           └─5190 gdm-session-worker [pam/gdm-launch-environment]
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
file /etc/opt/VirtualGL/vgl_xauth_key

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

Perform checks:
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


#### Step 01.03: Configure remote client machine.

**Important:** You will also need to create `vgluser` group with the same `gid` on every client
system that needs to access the remote server and add the same user on the client
machines to the `vglusers` group:
```bash
# create group vglusers
sudo addgroup --gid 1001 vglusers

# add user to vglusers group
sudo adduser developer vglusers
```

Now, using ssh to log in from another client machine and run the following
commands to test the VirtualGL configuration:
```bash
# environment variables
HOST_ADAPTER1_IP="192.168.1.11"
USER="developer"

# login
ssh -XY $USER@$HOST_ADAPTER1_IP

# check display
echo $DISPLAY
localhost:10.0

echo $VGL_DISPLAY
VGL_localhost:10.0

# get display info
xdpyinfo -display :10.0
/opt/VirtualGL/bin/glxinfo -display :10.0 -c

# check if access to the 3D X server is restricted
xauth merge /etc/opt/VirtualGL/vgl_xauth_key
xdpyinfo -display :0
/opt/VirtualGL/bin/glxinfo -display :0 -c

# check that the graphic card is indeed used and it is not using mesa

# now you can open up a graphic application from the remote machine,
# try "gedit" and all should go along smoothly.
```


### Step 02.00: Use cases.

#### Step 01.01: Make a VirtualGL connection.

Verify that X11 client forwarding is enabled:
```bash
ssh -X $USER@$HOST_ADAPTER1_IP xeyes
```

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
VGL_LOGO=1 vglrun -d :1 glxgears

# debug
VGL_TRACE=1 vglrun +v -d :1 glxgears
```

Run the `/opt/VirtualGL/bin//glxspheres64` performance test:
```bash
vglrun -d :1 glxspheres64
```

![virtualgl-glxspheres64.png](./image/virtualgl-glxspheres64.png)

#### Step 02.02: Make a TurboVNC connection.

This is more a complex (and a bit faster) solution for forwarding whole X sessions. More people can collaborate on the same session and it will last even after you close your connection, so you can continue with your work.

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

You can disconnect now.
```bash
exit
```

There are two options to connect to the remote server:
- directly use `vncviewer --extssh=1` which will automatically create an ssh tunnel and do port forwarding
- create an ssh tunnel manually and connect to local host using `vncviewer --extssh=1 localhost:1`

**Option 01:** Use vmcviewer to automatically create an ssh tunnel.

Type the following command to launch the vncviwer gui and specify `$HOST_ADAPTER1_IP:<DISPLAY>` in the VNC Server field.
```bash
# launches vncviwer gui
vncviewer --extssh=1
```

![turbovnc-new-connection-vnc-server.png](./image/turbovnc-new-connection-vnc-server.png)



**Option 02:** Create an ssh tunnel and connect to local host using vncviewer.

On your local machine, create an ssh tunnel for encrypting the VNC stream by typing
```bash
shh -N -q -L 1047:localhost:<display_number> user@server
```
where 1047 is the arbitrary free port number and <display_number> is obtained from previous the paragraph,
e.g.
```bash
HOST_ADAPTER1_IP="192.168.1.11"
USER="developer"

# run ssh tunnel in the foreground
ssh -N -q -L 1047:localhost:5901 $USER@$HOST_ADAPTER1_IP

# run ssh tunnel in the background
ssh -L 1047:localhost:5901 -N -f -l $USER@$HOST_ADAPTER1_IP

# to list running ssh processes
ps -aux | grep ssh
```

Now you can start the TurboVNC client on your local machine e.g. running command
```bash
vncviewer --extssh=1 localhost:1
```

If you cannot find the vncviewer command, this might help
```bash
/opt/TurboVNC/bin/vncviewer localhost:1047
```

Now you just need to type your chosen password for VNC and start the X-session,

Start the 3D accelerated application with vglrun command
```bash
vglrun application
```

e.g. vglrun glxinfo | more to see OpenGL status or ParaView vglrun paraview. (same as in the Scenario A)

---

## Troubleshooting

01. '[VGL] ERROR: Could not open display :0.'

That error typically means one of the following:

(1) You haven't run `vglserver_config`, so the 3D X server will not allow
access while it's sitting at the login prompt.

(2) You ran `vglserver_config` and configured the system such that only
users of the `vglusers` group can access the 3D X server, but you forgot
to add yourself to the `vglusers` group-- or you didn't log out and back
in so the new group permissions could take effect.

(3) There is no X server running on `display :0 `(but if you are able to
successfully use the system from within TurboVNC, then this must not be
the case.)

02. VirtualGL command line options:

VirtualGL options (see documentation for a more comprehensive list)

-c <c>    : proxy = Send rendered frames uncompressed using X11 Transport
                    [default if the 2D X server is on this machine]
            jpeg = Compress rendered frames using JPEG/send using VGL Transport
                   [default if the 2D X server is on another machine]
            rgb = Encode rendered frames as RGB/send using VGL Transport
            xv = Encode rendered frames as YUV420P/send using XV Transport
            yuv = Encode rendered frames as YUV420P/send using the VGL
                  Transport and display on the client using X Video
            [If an image transport plugin is being used, then <c> can be any
             number >= 0 (default = 0).]

-nodl     : Don't interpose the dlopen() function.  dlopen() is normally
            interposed in order to force applications that load libGL using
            dlopen() to load the VirtualGL Faker instead.  For the more common
            case of applications that link directly with libGL, disabling the
            dlopen() interposer makes VirtualGL less intrusive, since it will
            no longer load libGL until the 3D application actually uses that
            library.

-d <d>    : <d> = the X display/screen to use for 3D rendering [default = :0.0]

-fps <f>  : Limit image transport frame rate to <f> frames/sec

-gamma <g>: Set gamma correction factor to <g> (see User's Guide)

-ge       : Fool 3D application into thinking that LD_PRELOAD is unset

-ms <s>   : Force OpenGL multisampling to be enabled with <s> samples
            (<s> = 0 forces multisampling to be disabled)

-np <n>   : Use <n> threads to perform image compression [default = 1]

+/-pr     : Enable/disable performance profiling output [default = disabled]

-q <q>    : Compression quality [1 <= <q> <= 100]
            [default = 95 for JPEG/VGL Transport.  Has no effect with
             other built-in image transports or encoding types]

-samp <s> : Chrominance subsampling factor
            <s> = gray, 1x, 2x, 4x
            [default = 1x for JPEG/VGL Transport.  Has no effect with
             other built-in image transports or encoding types]

+/-s      : Enable/disable SSL encryption of VGL Transport or custom
            image transport, if applicable.
            [default = disabled.  Has no effect on the VGL Transport unless
             VirtualGL was built with OpenSSL support]

+/-sp     : Turn on/off frame spoiling [default = enabled]

-st <s>   : left = Read back/transport only left eye buffer of rendered stereo
                   frame
            right = Read back/transport only right eye buffer of rendered
                    stereo frame
            quad = Use quad-buffered stereo if available, otherwise use
                   red/cyan (anaglyphic) stereo [default]
            rc = Always use red/cyan (anaglyphic) stereo
            gm = Always use green/magenta (anaglyphic) stereo
            by = Always use blue/yellow (anaglyphic) stereo
            i = Always use interleaved (passive) stereo
            tb = Always use top/bottom (passive) stereo
            ss = Always use side-by-side (passive) stereo

+/-sync   : Enable/disable strict 2D/3D synchronization [default = disabled]

+/-tr     : Enable/disable function call tracing (generates a lot of output)
            [default = disabled]

-trans <t>: Use transport plugin contained in library libvgltrans_<t>.so

+/-v      : Enable/disable verbose VirtualGL messages [default = disabled]

+xcb/-xcb : Enable/disable XCB interposer [default = enabled]

+wm/-wm   : Enable/disable window manager mode (for running compiz, etc.)


Example:
```bash
vglrun -c jpeg -np 32 +s -d :1 glxgears
```

---

## Related Topics

01.[vgl transport: '[VGL] ERROR: Could not open display :0.'](https://virtualgl-users.narkive.com/kUckmG7X/vgl-transport-vgl-error-could-not-open-display-0)
