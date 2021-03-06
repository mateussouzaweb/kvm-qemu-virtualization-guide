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

- ``--main-gpu-passthrough``: Enables **single GPU passthrough** support in KVM/QEMU virtual machines, by automatically attach/detach the GPU from the host and dynamically attach it to the VM and vice versa (please notice that this is required only if your setup contains only one GPU. This also will end the current host display session, requiring login again after VM has been stopped).
- ``--gpu-reset=$ID``: Define the reset method of the referenced PCI device to *device_specific*. This is required if you have an AMD GPU with vendor reset bug on kernel 5.15+.

CPU:

- ``--scale-cpu``: Sets the CPU scaling governor as *performance* mode on the host system to increase performance of running virtual machines. Restore on VM shutdown.
- ``--cpu-host-allow=$CORES``: Set CPU pinning when the VM is being started. You should set only the host allowed CPU cores (cores not used by the VM).
- ``--cpu-host-restore=$CORES``: Restore CPU pinning on VM shutdown. In most of the use cases, all system CPU cores should be restored.

USB:

- ``--usb-passthrough=$DEVICE``: Allow live USB passthrough of any USB device to the VM. This also reduce the need of declare USB devices on the VM XML file. This option can be used multiple times to specify custom devices. Valid device values are ``all``, ``BUS:$NUMBER`` for specific bus, ``DEV:$BUS:$NUMBER`` for specific device number on bus or `ID:$ID` for specific device ID.

## Installation

Use the process below to create and install the QEMU hooks (you can study the code to learn more if you are a programmer or sysadmin):

```bash
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Scripts/hooks"
DESTINATION="/etc/libvirt/hooks"

mkdir -p ${DESTINATION} ${DESTINATION}/udev
curl -L ${REPOSITORY}/qemu --output ${DESTINATION}/qemu
curl -L ${REPOSITORY}/udev/usb --output ${DESTINATION}/udev/usb
```

We also need to attach USB rules to passthrough USB devices:

```bash
# Add rule
echo 'SUBSYSTEM=="usb",RUN+="'${DESTINATION}'/udev/usb"' > /etc/udev/rules.d/90-libvirt-usb.rules

# Reload rules config
udevadm control --reload-rules
```

Done! Configure the VM with the desired options and you are good to go.

----

Next step: **[Managing Virtual Machines](4%20-%20Management.md)**
