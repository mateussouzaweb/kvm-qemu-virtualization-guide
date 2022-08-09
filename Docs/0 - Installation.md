# Host System Installation

Based on my own experience, after many tries in various distros, Fedora Server is the most stable distribution with blinding edge features where things simply just work. The nature of its terminal only environment also make easier to passthrough devices while avoid common issues.

Other distributions always resulted in problems for me or got a process more complicated. Some don't have some packages yet by default and many other issues... I decided to focus on Fedora Server because of the need of a safe environment (I don`t want to start from scratch every time that a package break my host system), but that should work on Fedora Workstation as well. Give it a try! You can download the ISO at <https://getfedora.org/server/download/>

## How to Install

First, install the operational system in your machine if it has not been done yet.

If you are not using the **root** user, remember to use ``sudo`` (or ``sudo su``) on the commands of this process and after each reboot.

If you have another machine and can connect to it via SSH, that would be great too, copy and paste are much faster than manual typing these commands.

After the OS is installed, run the following commands to update the system and install basic packages:

```bash
# Update system packages
dnf update -y

# Install the basic packages
dnf install -y vim wget \
    curl openssl \
    zip htop \
    git git-core \
    terminus-fonts-console \
    python3 python3-pip
```

Now, install virtualization packages:

```bash
# Virtualization packages
dnf install -y \
    qemu-kvm \
    libvirt \
    virt-install \
    libguestfs-tools \
    cockpit \
    cockpit-machines

# Make sure permissions are correct
/sbin/restorecon -R -vF /var/lib/libvirt

# Enable the service
systemctl enable --now libvirtd 
systemctl enable --now cockpit.socket

# Enable firewall
firewall-cmd --add-service=cockpit
firewall-cmd --add-service=cockpit --permanent
```

If you are using a full-fledged desktop, also install the GUI software:

```bash
# Desktop mode ONLY
dnf install -y libgovirt virt-viewer virt-manager
```

For setups with other users that are not the **root** user, you also need to make sure users always connect to QEMU system URI because only this endpoint has access to the host resources. For each user that will run VMs, run the following commands:

```bash
# Update with correct username
USERNAME="mateussouzaweb"

# Apply group permissions to the user
usermod -a -G libvirt,kvm ${USERNAME}

# Create config directory and append configuration
DESTINATION="/home/${USERNAME}/.config/libvirt"
mkdir -p ${DESTINATION}
echo 'uri_default = "qemu:///system"' >> "${DESTINATION}/libvirt.conf"
```

Reboot and you are done:

```bash
reboot
```

Congratulations!

You can now access the web UI of Cockpit with another machine for remote management with the address format ``https://$IP:9090/``. For the desktop mode, you can also use the Virt Manager application to manage VMs.

After this process, you should have the necessary packages to run basic virtualization and we can proceed to tweak the system for full and greater virtualization!

----

Next step: **[Configuring the Bridge Network](1%20-%20Bridge%20Network.md)**
