# Host System Installation

To start, you need an operational system for the host that will be the hypervisor. After many tries in various distros, Fedora Server and Fedora Workstation was the most stable distribution with blinding edge features where things simply just work, so to start, I strongly recommend using Fedora as the hypervisor. Other distributions are great, but always resulted in problems for me or got a process more complicated and since we are focusing here on a concrete and working solution, Fedora was obviously the alternative.

I also decided to focus on Fedora Server first because the nature of its terminal only environment makes it easier to passthrough devices to VMs and avoid common issues available in full desktop environments. The guide should work on Fedora Workstation as well and relevant information will be available when necessary.

First, install the operational system in your machine if it has not been done yet. You can download the ISO at <https://getfedora.org> if you decide to use Fedora. To guide you in this process, here a few general recommendations:

- If you have another machine and can connect to it via SSH, that would be great too, copying and pasting are much faster than manual typing these commands.

## Installing Virtualization Support

After the OS is installed, run the following commands to update the system and install basic packages used in common scenarios:

```bash
# Update system packages
sudo dnf update -y

# Install the basic packages
sudo dnf install -y vim wget \
    curl openssl \
    zip htop lm_sensors \
    git git-core \
    terminus-fonts-console \
    python3 python3-pip
```

Now, install the virtualization packages:

```bash
# Virtualization packages
sudo dnf install -y \
    qemu-kvm \
    libvirt \
    virt-install \
    guestfs-tools \
    libguestfs-tools

# Make sure permissions are correct
sudo restorecon -R -vF /var/lib/libvirt

# Enable the service
sudo systemctl enable --now libvirtd
```

If you are using a full-fledged desktop, also install the GUI software. After installation, you can also use the Virtual Machine Manager application to manage VMs within your mouse:

```bash
# Desktop mode ONLY
sudo dnf install -y libgovirt virt-manager
```

If you want the hypervisor to be remote accessible, you should install the Cockpit package. After installation, you also can access the web UI of Cockpit with another machine for remote management with the address in the format ``https://$IP:9090/``:

```bash
# Remote access ONLY
sudo dnf install -y cockpit cockpit-machines cockpit-pcp

# Enable the service
sudo systemctl enable --now cockpit.socket

# Enable firewall
sudo firewall-cmd --permanent --add-service=cockpit
sudo firewall-cmd --reload
```

For setups with other users that are not the **root** user, you also need to make sure users always connect to QEMU system URI because only this endpoint has access to the host resources. For each user that will run VMs, run the following commands:

```bash
# Update with correct username
USERNAME="mateussouzaweb"

# Apply group permissions to the user
sudo usermod -a -G libvirt,kvm ${USERNAME}

# Create config directory and append configuration
DESTINATION="/home/${USERNAME}/.config/libvirt"
sudo mkdir -p ${DESTINATION}
sudo sh -c "echo 'uri_default = \"qemu:///system\"' >> \"${DESTINATION}/libvirt.conf\""
sudo chown -R ${USERNAME}:${USERNAME} ${DESTINATION}
```

Reboot and you are done:

```bash
sudo reboot
```

Congratulations!

You now have the necessary packages to run basic virtualization and we can proceed to tweak the system for full and greater virtualization!

----

Next step: **[Configuring the Bridge Network](01%20-%20Bridge%20Network.md)**
