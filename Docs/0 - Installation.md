# Host System Installation

Based on my own experience, after many tries in various distros, Fedora Server is the most stable distribution with blinding edge features where things simply just work. The nature of its terminal only environment also make easier to passthrough devices while avoid common issues.

Other distributions always resulted in problems for me or got a process more complicated. Some don't have some packages yet by default and many other issues... I decided to focus on Fedora Server because of the need of a safe environment (I don`t want to start from scratch every time that a package break my host system), but that should work on Fedora Workstation as well. Give it a try! You can download the ISO at <https://getfedora.org/server/download/>

## How to Install

Simply install the operational system in your machine if it has not been done yet.

I do recommend installing with **root** user only because this will prevent any access level issue that you can have. If you choose to have another user, remember to use ``sudo`` (or ``sudo su``) on the commands of this process.

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
    cockpit-machines

# Make sure permissions are correct
/sbin/restorecon -R -vF /var/lib/libvirt

# Enable the service
systemctl enable libvirtd --now
```

Reboot and you are done:

```bash
reboot
```

Congratulations!

You can now access the web UI of Cockpit with another machine for remote management with the address format ``https://$IP:9090/``.

After this process, you should have the necessary packages to run basic virtualization and we can proceed to tweak the system for full and greater virtualization!

----

Next step: **[Configuring the Bridge Network](1%20-%20Bridge%20Network.md)**
