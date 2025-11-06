# Virtualization Hooks for QEMU

KVM/QEMU contains a great feature called hooks that allow us to run temporary system modifications when a virtual machine is being started or stopped. We will use the hooks feature to increase the VMs ecosystem by enabling support for single GPU passthrough and other enhancements on the host.

Differently of others hook scripts for QEMU that you can see around the web, each feature will be configured on the XML details of each VM to allow us to attach and detach PCI-e devices and others system features based on the VM configurations. That way, we can create a single point of management for each VM that also doesn't mess up with setups containing multiple GPUs for example.

To activate these features on a specific VM, you must edit the VMs details and add the desired options on the ``<description>`` entry (create the entry if not exist):

```bash
virsh edit ${NAME}
```

Options are like command line options, so you must set all in one single line. After setting the desired configuration, just start the VM and everything should work. To disable one feature, simply remove the declaration previously added to the ``<description>`` entry.

## Available Options

The current list of available feature options is:

GPU:

- ``--gpu-passthrough=$ID,$VGA,$AUDIO``: Enables **GPU passthrough** support in KVM/QEMU virtual machines, by automatically attach/detach the GPU from the host and dynamically attach it to the VM and vice versa. This is all in one solution that works for **single GPU passthrough setups** or more. To passthrough the main GPU that is being used at your system, set ID as ``main``, otherwise you can set any ID for identification for example: ``nvidia``, ``amd``, ``secondary``... Please notice that if you set ID as ``main``, the hook will end the current host display session to free the GPU and you will be required to login again after VM has been stopped. Additionally, if your GPU needs vendor reset, append the additional fourth parameter as ``reset`` on this option. If you need to reduce your AMD GPU Resizable BAR before starting the VM, append the additional fourth parameter as ``resize-bar`` on this option. To passthrough multiple GPUs, just use this option again with the additional GPU details. 

CPU:

- ``--cpu-scaling-mode=$MODE``: Sets the CPU scaling governor mode. Set it as *performance* or *schedutil* to increase performance of running virtual machines. Automatically restores to the default scaling governor on VM shutdown.
- ``--preserve-cores=$CORES``: Set the cores on the CPU to be preserved to the host, while dedicating others cores to VMs. This is the same as setting the CPU pinning when the VM is being started and restore CPU pinning on VM shutdown.

USB:

- ``--usb-passthrough=$DEVICE``: Allow live USB passthrough of any USB device to the VM. This also reduce the need of declare USB devices on the VM XML file. This option can be used multiple times to specify custom devices. Valid device values are ``all``, ``BUS:$NUMBER`` for specific bus, ``DEV:$BUS:$NUMBER`` for specific device number on bus or `ID:$ID` for specific device ID.

## Installation

Use the process below to create and install the QEMU hooks (you can study the code to learn more if you are a programmer or sysadmin):

```bash
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Scripts/hooks"
DESTINATION="/etc/libvirt/hooks"

sudo mkdir -p ${DESTINATION} ${DESTINATION}/udev
sudo curl -L ${REPOSITORY}/qemu --output ${DESTINATION}/qemu
sudo curl -L ${REPOSITORY}/udev/usb --output ${DESTINATION}/udev/usb

sudo chmod +x ${DESTINATION}/qemu
sudo chmod +x ${DESTINATION}/udev/usb
sudo restorecon -R -vF ${DESTINATION}
```

We also need to attach USB rules to passthrough USB devices:

```bash
# Add rule
sudo sh -c "echo 'SUBSYSTEM==\"usb\",RUN+=\"'${DESTINATION}'/udev/usb\"' > /etc/udev/rules.d/90-libvirt-usb.rules"

# Reload rules config
sudo udevadm control --reload-rules
```

Done! Configure the VM with the desired options and you are good to go.

## Debugging

If you are interested in knowing what is going on when these hooks are executed, just watch for the logs. This also can be useful to detect issues with your setup:

```bash
sudo dmesg -w
sudo tail -f /var/log/libvirt/qemu/${NAME}.log
```

----

Next step: **[Managing Virtual Machines](04%20-%20Virtual%20Machine%20Management.md)**
