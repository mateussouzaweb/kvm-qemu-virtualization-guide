# Virtualization Hooks for QEMU

KVM/QEMU contains a great feature called hooks that allow us to run temporary system modifications when a virtual machine is being started or stopped. We will use the hooks feature to increase the VMs ecosystem by enabling support for single GPU passthrough and other enhancements on the host.

Differently of others hook scripts for QEMU that you can see around the web, this process is now automated by my additional project with scripts for libvirt hooks that does everything for you. Please take a look on the project page if you want to know more: <https://github.com/mateussouzaweb/libvirt-hooks>

## Hooks Installation

The installation of hooks is very easy. Use the process below to install the script:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Set repository and destination
REPOSITORY="https://github.com/mateussouzaweb/libvirt-hooks/"
DESTINATION="/etc/libvirt/hooks"

# Download binary file
sudo mkdir -p ${DESTINATION}
sudo curl -L ${REPOSITORY}/releases/latest/download/qemu-amd64 \
    --output ${DESTINATION}/qemu

# Set scripts as executable
sudo chmod +x ${DESTINATION}/qemu

# Install udev rules
sudo ${DESTINATION}/qemu install
```

When using Fedora, just make sure that permissions are correct for the script:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER / ATOMIC:**

```bash
# Fedora ONLY
# Make sure SElinux permissions are correct
sudo restorecon -R -vF ${DESTINATION}
```

Done! Create and configure your VM and you are good to go - yes, is that easy.

## Debugging

If you are interested in knowing what is going on when these hooks are executed, just watch for the system logs. This also can be useful to detect issues with your setup:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
sudo dmesg -w
```

----

Next step: **[Managing Virtual Machines](04%20-%20Virtual%20Machine%20Management.md)**
