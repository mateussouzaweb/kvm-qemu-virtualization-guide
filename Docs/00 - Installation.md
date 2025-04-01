# Host System Installation

First, we need an solid operational system for the host that will be the hypervisor.

In my experience, Fedora is the most stable distribution with blinding edge features where things simply just works, and I strongly recommend you start using Fedora as the hypervisor. Today, Fedora Project is separated on many variations that also makes him an interesting choose as the hypervisor since you can choose between traditional systems like Fedora Server and Fedora Workstation or immutables systems with Fedora Atomic Desktop.

How about others Linux distributions? Many of them are great, but when we are speaking about hypervisor for virtualization, they always resulted in problems for me or got a process more complicated and since we are focusing here on a concrete and working solution, Fedora is obviously the best alternative - you can download Fedora at <https://getfedora.org> and install it on your machine if it has not been done yet.

---

Before we start with the virtualization process, here are a few information and general recommendations:

- The guide will contain instructions for both traditional and immutable systems and will be provided as CLI commands.
- Additional instructions and relevant information about specific environments will be available when necessary.
- If you decided for Fedora Server, you should connect to it via SSH, because copying and pasting are much faster than manual typing these commands.
- If you decided for Fedora Workstation, please ensure to follow extra steps for desktop environments.
- If you decided for Fedora Atomic Desktops, is important to notice that the process is slightly different and specific commands will be provided as well.

## Installing Virtualization Support

After the OS is installed, run the following commands on traditional systems to update the OS, install basic packages used in common scenarios and install the virtualization packages:

```bash
# Update system packages
sudo dnf update -y

# Install the basic packages
sudo dnf install -y \
    vim wget curl openssl zip htop lm_sensors git \
    terminus-fonts-console python3 python3-pip

# Install virtualization packages
sudo dnf install -y \
    qemu-kvm libvirt virt-install \
    guestfs-tools libguestfs-tools bridge-utils
```

For immutable systems, you should run these commands, which will do the same in their way:

```bash
# Update system and reboot
rpm-ostree update
systemctl reboot

# Install basic packages in OSTree layer
rpm-ostree install -y \
    vim wget curl openssl zip htop lm_sensors git \
    terminus-fonts-console python3 python3-pip

# Install virtualization packages in OSTree layer
rpm-ostree install -y \
    qemu-kvm libvirt virt-install  \
    guestfs-tools libguestfs-tools bridge-utils
```

If you are using a full-fledged desktop environment, you can also use the Virtual Machine Manager application to manage VMs with your mouse and keyboard from the GUI. Here is the instruction to install the GUI software:

```bash
# Desktop mode ONLY
sudo dnf install -y libgovirt virt-manager

# Immutable OS / Atomic Desktop ONLY
# Install packages in OSTree layer
rpm-ostree install -y libgovirt virt-manager
```

If you want the hypervisor to be remote accessible, you should install the Cockpit package. After installation, you also can access the web UI of Cockpit with another machine for remote management with the address in the format ``https://$IP:9090/``:

```bash
# Remote access ONLY
sudo dnf install -y cockpit cockpit-machines

# Immutable OS ONLY
# Install packages in OSTree layer
rpm-ostree install -y cockpit cockpit-machines cockpit-ostree
```

Finally, reboot and you are done:

```bash
sudo reboot
```

The reboot process is necessary to ensure that packages are enabled for immutable systems in a new OSTree layer, but is also a good practice for traditional desktops. Once the system has been rebooted, you can proceed to enabling virtualization.

## Enabling Virtualization

Now that all the necessary packages are installed, you just need to enable the services:

```bash
# Make sure permissions are correct
sudo restorecon -R -vF /var/lib/libvirt

# Enable the libvirt service
sudo systemctl enable --now libvirtd

# Immutable OS ONLY
# Add virtualization groups to persistent file
grep -E '^libvirt:' /usr/lib/group | sudo tee -a /etc/group
grep -E '^kvm:' /usr/lib/group | sudo tee -a /etc/group
```

If you installed Cockpit for remote management, you also need to ensure it has been enabled and firewall rules where added to the system:

```bash
# Enable the cockpit service
sudo systemctl enable --now cockpit.socket

# Enable firewall for cockpit access
sudo firewall-cmd --permanent --add-service=cockpit
sudo firewall-cmd --reload
```

For setups with other users that are not the **root** user, you also need to make sure users always connect to QEMU system URI because only this endpoint has access to the host resources. For each user that will run VMs, run the following commands:

```bash
# Update with correct username
USERNAME="mateussouzaweb"

# Apply group permissions to the user
sudo usermod -aG libvirt,kvm ${USERNAME}

# Create config directory and append configuration
DESTINATION="/home/${USERNAME}/.config/libvirt"
sudo mkdir -p ${DESTINATION}
sudo sh -c "echo 'uri_default = \"qemu:///system\"' >> \"${DESTINATION}/libvirt.conf\""
sudo chown -R ${USERNAME}:${USERNAME} ${DESTINATION}
```

Congratulations!

You now have the necessary packages to run basic virtualization and we can proceed to tweak the system for full and greater virtualization!

----

Next step: **[Configuring the Bridge Network](01%20-%20Bridge%20Network.md)**
