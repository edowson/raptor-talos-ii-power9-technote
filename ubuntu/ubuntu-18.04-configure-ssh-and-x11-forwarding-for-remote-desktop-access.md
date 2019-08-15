# Ubuntu-18.04 - Configure SSH and X11 forwarding for remote desktop access

## Procedure

### Step 01.00: Setup SSH for remote access

#### Step 01.01: Install OpenSSH

Type the following commands to install OpenSSH.

```bash
sudo apt-get install openssh-client openssh-server
```

#### Step 01.02: Generate RSA Keys for SSH users

Environment variables:
```bash
USER='developer'
UID='1002'
```

Generate RSA keys for user:

```bash
su $USER
cd ~/
ssh-keygen -t rsa -b 8192

# copy the public key
cp id_rsa.pub $USER@talos2.pub
```

Restart ssh:

```bash
sudo service ssh restart
```

Login using ssh in debug mode:

```bash
ssh -vv $USER@talos2
```

Check the ssh logs for info on failed login attempts:

```bash
cat /var/log/auth.log
```

#### Step 04.03: Configure SSH

Create a list of authorized users:

```bash
cd ~/.ssh
touch authorized_keys
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
```

Ensure that id_rsa is only accessible by the current user:
```bash
cd ~/.ssh
chmod go-r id_rsa
```

Append additional users to the list of authorized users on host domain:

```bash
cd ~/.ssh
cat $USER@apollo.pub >> authorized_keys
chmod 600 authorized_keys
```

Make a backup of your sshd_config file by copying it to your home directory, or by making a read-only copy in /etc/ssh by doing:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.factory-defaults
sudo chmod a-w /etc/ssh/sshd_config.factory-defaults
```

Modify the sshd_config file

```bash
sudo gedit /etc/ssh/sshd_config
```

Disable root login.

```
PermitRootLogin no
```

[optional] Change the SSH port to something different than the standard port 22.

```
Port 8420
```

Disable tunneled clear text password authentication. See SSH/OpenSSH/Configuring - Community Ubuntu Documentation for more details.

To disable password authentication, look for the following line in your sshd_config file:

```
#PasswordAuthentication yes
```

replace it with a line that looks like this:

```
PasswordAuthentication no
```

Edit `/etc/ssh/sshd_config` to enable X11 forwarding and disable using the X11 local host:
```bash
X11Forwarding yes
X11UseLocalhost no
```

**Important:** If `X11UseLocalhost` is not set to `no`, you will get the following errors when connecting to the remote server from a remote client:
```bash
X11 connection rejected because of wrong authentication.
Error: Cant open display: localhost:10.0
```


Make sure your local ssh_config has following lines:
```bash
Host *
ForwardX11 yes
```

Verify X11 client forwarding is enabled:
```bash
grep X11Forwarding /etc/ssh/sshd_config
X11Forwarding yes
#	X11Forwarding no

grep X11 /etc/ssh/ssh_config
ForwardX11 yes
#   ForwardX11Trusted yes
```

Modify /etc/ssh/ssh_config:
```bash
sudo nano /etc/ssh/ssh_config
```

Change `ForwardX11` to `yes`.


Restart the ssh server:

```bash
sudo service ssh restart
```

Once you have saved the file and restarted your SSH server, you shouldn't even be asked for a password when you log in.

Add the list of servers to your /etc/hosts file

```bash
$ sudo gedit /etc/hosts

##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost

# This server's hostname and FDQN.
192.168.1.11    talos2   # Ubuntu-18.04 64-bit physical machine.
```

Verify that X11 client forwarding is enabled:
```bash
ssh -X $USER@$HOST_ADAPTER1_IP xeyes
```


#### Step 04.04: SSH Hardening

Harden the server using the instructions outlined in How to secure an Ubuntu 12.04 LTS server - Part 1 The Basics | The Fan Club.

SSH Hardening - disable root login and change port.

The easiest way to secure SSH is to disable root login and change the SSH port to something different than the standard port 22. Before disabling the root login create a new SSH user and make sure the user belongs to the admin group (see step 4. below regarding the admin group).

Protect su by limiting access only to admin group.

```bash
sudo groupadd admin
sudo usermod -aG admin $USER
sudo dpkg-statoverride --update --add root admin 4750 /bin/su
```

Restart the ssh server:

```bash
sudo service ssh restart
```

[Optional: ]If you change the SSH port also open the new port you have chosen on the firewall and close port 22.

```bash
sudo ufw enable
sudo ufw status
sudo ufw default deny
sudo ufw deny 22
sudo ufw allow 8420/tcp
```

See UFW - Community Ubuntu Documentation, for more details.

To test if everything is working correctly, try to login using the new ssh port

```bash
ssh -p 8420 $USER@talos2
```

To clone a repo, using the following command syntax

```bash
git clone ssh://git@remote-server:[port]/[path-to-git-repository]
```

To test the ssh connection, use the ssh command with the verbose option:

```bash
ssh -p 8420 -vv $USER@talos2
```

To transfer files using `scp` between two computers, type the following command:

```bash
# copy a single file
scp $USER@apollo.pub $USER@192.168.1.11:/home/$USER/.ssh

# specify ssh port
scp -P 8420 <filename> $USER@talos2:/home/$USER/Downloads

# recursively copy the contents of an entire folder
scp -r -P 22 <folder> $USER@talos2:/home/$USER/Downloads
```
