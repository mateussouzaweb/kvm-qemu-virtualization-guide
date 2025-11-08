# Host System Installation

You need a good operational system for the host that will be the hypervisor. I will present here a few options that are compatible with this guide:

- Fedora Server 40+
- Fedora Workstation / KDE 40+
- Fedora Atomic Desktop 40+ (also know as immutable OS)
- Ubuntu Server 26.04 LTS+
- Ubuntu Desktop 26.04 LTS+

In my experience, Fedora is the most stable distribution with blinding edge features where things simply just works, and I strongly recommend you start using Fedora as the hypervisor. Today, Fedora Project is separated on many variations that also makes him an interesting choose as the hypervisor since you can choose between traditional systems like Fedora Server, Fedora Workstation and Fedora KDE or immutables systems with Fedora Atomic Desktop.

The guide also includes support for Ubuntu Desktop and Ubuntu Servers starting from 26.04 LTS due to the recent switch to Dracut to manage the initial boot process. If you don't like how Fedora handles Nvidia drivers or multimedia codecs, Ubuntu is a solid alternative with long term support for those who want stability for years.

How about others Linux distributions? Many of them are great, but when we are speaking about hypervisor for virtualization, they always resulted in problems for me or got a process more complicated and since we are focusing here on a concrete and working solution, Fedora and Ubuntu are obviously the best alternative.

Before we start with the virtualization process, here are a few information and general recommendations:

- The guide will contain instructions for both traditional and immutable systems and will be provided as CLI commands.
- This guide tries to be clear about what commands you should run based on your operating system choose. Please pay attention of what you are doing and don't run all commands available in the page.
- Additional instructions and relevant information about specific environments will be available when necessary.
- If you decided for operating system in server mode, you should connect to it via SSH, because copying and pasting are much faster than manual typing these commands.
- If you decided for any operating system with desktop mode, please ensure to follow extra steps for desktop environments .
- If you decided for immutable or atomic operating system, is important to notice that the process is slightly different and specific commands will be provided as well.

## Installing Virtualization Support

After the OS is installed in your machine, run the following commands to update the OS, install basic packages used in common scenarios and the virtualization packages.

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora ONLY
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

![Fedora](../Images/fedora.png)
**FEDORA - ATOMIC:**

```bash
# Fedora Immutable ONLY
# Update system and reboot
rpm-ostree update
systemctl reboot

# Install basic packages in OSTree layer
rpm-ostree install -y \
    vim htop lm_sensors git python3-pip terminus-fonts-console

# Install virtualization packages in OSTree layer
rpm-ostree install -y \
    qemu-kvm libvirt virt-install  \
    guestfs-tools libguestfs-tools bridge-utils
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu ONLY
# Update system packages
sudo apt update
sudo apt upgrade -y
sudo apt autoremove

# Install the basic packages
sudo apt install -y \
    vim wget curl openssl libssl-dev zip htop git \
    lm-sensors python3 python3-pip

# Install virtualization packages
sudo apt install -y \
    qemu-kvm libvirt-clients-qemu libvirt-daemon-system \
    virtinst guestfs-tools libguestfs-tools \
    bridge-utils ovmf
```

## Virtualization Desktop Interface

If you are using a full-fledged desktop environment, you can also use the *Virtual Machine Manager* application to manage VMs with your mouse and keyboard from the GUI. Here you will find the instruction to install the GUI software for usage on desktop environments.

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP:**

```bash
# Fedora Desktop ONLY
sudo dnf install -y libgovirt virt-manager
```

![Fedora](../Images/fedora.png)
**FEDORA - ATOMIC DESKTOP:**

```bash
# Fedora Immutable Desktop ONLY
# Install packages in OSTree layer
rpm-ostree install -y libgovirt virt-manager
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP:**

```bash
# Ubuntu Desktop ONLY
sudo apt install -y virt-manager
```

## Virtualization Remote Management

If you want the hypervisor to be remote accessible to manage the system and virtualization, you should install the *Cockpit* package. When installed, you will be able to access the web UI of Cockpit with another machine or from your phone for remote management by opening the browser with the address in the format ``https://$IP:9090/``:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora - Remote access ONLY
sudo dnf install -y cockpit cockpit-files cockpit-machines cockpit-podman
```

![Fedora](../Images/fedora.png)
**FEDORA - ATOMIC:**

```bash
# Fedora Immutable - Remote access ONLY
# Install packages in OSTree layer
rpm-ostree install -y cockpit cockpit-files cockpit-machines cockpit-podman cockpit-ostree
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu - Remote access ONLY
sudo apt install -y cockpit cockpit-machines python3-pcp

# Fix cockpit connection issues
# https://cockpit-project.org/faq#error-message-about-being-offline
sudo nmcli con add type dummy \
    con-name fake ifname fake0 \
    ip4 1.2.3.4/24 gw4 1.2.3.1

sudo bash -c "cat >> /etc/NetworkManager/conf.d/10-globally-managed-devices.conf" << EOL
[keyfile]
unmanaged-devices=none
EOL
```

## Reboot

Finally, reboot the system to complete the virtualization packages installation. The reboot process is necessary to ensure that packages are enabled for immutable systems in a new OSTree layer, but is also a good practice for traditional desktops:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Fedora / Ubuntu
sudo reboot
```

Once the system has been rebooted, you can proceed to enabling virtualization.

## Enabling Virtualization

Now that all the necessary packages are installed, you just need to enable the services in the operating system to start using it:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora ONLY
# Make sure permissions are correct
sudo restorecon -R -vF /var/lib/libvirt

# Enable the libvirt service
sudo systemctl enable --now libvirtd
```

![Fedora](../Images/fedora.png)
**FEDORA - ATOMIC:**

```bash
# Fedora Immutable ONLY
# Make sure permissions are correct
sudo restorecon -R -vF /var/lib/libvirt

# Enable the libvirt service
sudo systemctl enable --now libvirtd

# Add virtualization groups to persistent file
grep -E '^libvirt:' /usr/lib/group | sudo tee -a /etc/group
grep -E '^kvm:' /usr/lib/group | sudo tee -a /etc/group
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu ONLY
# Enable the libvirt service
sudo systemctl enable --now libvirtd
```

If you installed Cockpit for remote management, you also need to ensure it has been enabled and are accessible with firewall rules where added to the system:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER / ATOMIC:**

```bash
# Fedora ONLY
# Enable firewall for cockpit access
sudo firewall-cmd --permanent --add-service=cockpit
sudo firewall-cmd --reload

# Enable the cockpit service
sudo systemctl enable --now cockpit.socket
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu ONLY
# Enable firewall for cockpit access
sudo ufw allow 9090

# Enable the cockpit service
sudo systemctl enable --now cockpit.socket
```

For setups with other users that are not the **root** user, you also need to make sure users always connect to QEMU system URI because only this endpoint has access to the host resources. For each user that will run VMs, run the following commands:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Fedora / Ubuntu
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
