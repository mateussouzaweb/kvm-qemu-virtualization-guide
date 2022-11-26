# MacOS Virtualization - Tips and Tricks

MacOS has a few tricks to virtualize. Once running with their limitations, it works great and can be used as a daily machine.

## Observations

**Installation:**

- Like Linux VMs, installation process does not require virtualized display graphics if you start with the correct boot flag for AMD GPUs in OpenCore config.
- Otherwise, enable the virtualization display graphics and proceed.

**QEMU:**

- Don't forget to include the QEMU XML namespace on top of the file.
- QEMU command line extra parameters are required to set correct flags for MacOS virtualization.

**OS:**

- Machine type must be ``pc-q35-4.2``, otherwise installation won't work.
- You must use the custom patched files ``OVMF_CODE.fd`` and ``OVMF_VARS.fd``.

**CPU:**

- If you have an AMD CPU, don't worry, it will work.
- Nested virtualization on AMD CPUs is not possible. This will limit the development environment in some cases. 
- If your CPU is a Mac compatible Intel chip, change ``Cascadelake-Server`` to ``host`` on the XML settings.

**Graphics:**

- GPU passthrough will not work on recent Nvidia graphic cards, forget it and buy a compatible AMD GPU.
- If you will passthrough any graphic card, just don't forget to remove the default video interface. Also make sure the gfx and audio devices are in the same bus (like ``0x01``) but in different function (``0x00`` and ``0x01``).
- For AMD GPU, you also need to set ``agdpmod=pikera`` in the ``config.plist`` NVRAM boot-args section - this file is available on the OpenCore EFI partition.

**Network:**

- Network interface model type must be ``vmxnet3``.
- Make sure the address is in bus ``0x0`` and slot ``0x0y`` (y is numeric).
- These settings will solve any issues that you can have on the installation process, built-in NIC, Apple Store and iCloud.

**Audio:**

- If you will passthrough onboard audio, make sure you put it in bus ``0x00`` and slot ``0x0y`` (y is numeric).
- This is necessary to make AppleALC recognize its audio device.

**USB:**

- Live USB passthrough is not possible. You need to attach the USB device and then restart the VM to work.
- Attached USB keyboard at start may not work on OpenCore boot menu. To solve this, you must passthrough the USB Controller device.
- USB passthrough of USB 3.X devices may not work on MacOS.

**Boot:**

- If the keyboard does not work on OpenCore boot menu, you can use the Console to select the boot entry. Just enter the Console and press the arrow key to select the disk entry (no need to attach VNC graphics for it).
- If the default boot entry is on the recovery image, you can change it by selecting the MacOS volume and hitting ``CTRL`` + ``Enter`` to set this as the default boot entry. The change will be reflected in the next boot.

**Post Install:**

- Download the apps: Hackintool, ProperTree and GenSMBIOS to modify OpenCore.
- Generate a unique SMBIOS and configure it in your EFI/OpenCore partition.
- Install additional kexts for your hardware if required.

**Sample XMLs:**

- You can check some configurations for MacOS machines in the ``Samples/`` folder.
- Install instructions are available below.

## Installation

See: <https://github.com/kholia/OSX-KVM>

This guide uses the OSX-KVM project as base system. We grab just the necessary files to run it on QEMU. Start by downloading files and fetching the desired MacOS version:

```bash
# Download files
REPOSITORY="https://github.com/kholia/OSX-KVM/raw/master"
wget -O /usr/share/OVMF/MACOS_CODE.fd ${REPOSITORY}/OVMF_CODE.fd
wget -O /usr/share/OVMF/MACOS_VARS.fd ${REPOSITORY}/OVMF_VARS.fd
wget -O /var/lib/libvirt/images/fetch-macOS-v2.py ${REPOSITORY}/fetch-macOS-v2.py
wget -O /var/lib/libvirt/images/opencore.qcow2 ${REPOSITORY}/OpenCore/OpenCore.qcow2

# Fetch MacOS
chmod +x /var/lib/libvirt/images/fetch-macOS-v2.py
cd /var/lib/libvirt/images
./fetch-macOS-v2.py

# Convert DMG image to IMG format and remove unnecessary files
qemu-img convert BaseSystem.dmg -O raw macos-{VERSION}.img
rm -f BaseSystem.dmg BaseSystem.chunklist

# Fix permissions
chmod -R 660 /var/lib/libvirt/images/*
chown -R qemu:qemu /var/lib/libvirt/images
/sbin/restorecon -R -vF /var/lib/libvirt/images
/sbin/restorecon -R -vF /usr/share/OVMF
```

Now, download the sample XML configuration and tweak to run on your system devices:

```bash
# Download sample
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Samples"
curl -L ${REPOSITORY}/macos.xml --output macos.xml

# Edit settings
vim macos.xml
```

Here, you can also tweak the OpenCore partition before starting the VM installation directly from the host (see the guide below). I will do it to enable the AMD GPU support for example, but if you don't know what to do, simply continue this guide...

Finally define the VM and start it:

```bash
virsh define macos.xml
virsh start macos
```

Done!

## Editing OpenCore from the Hypervisor

If you want to mount the OpenCore partition outside the MacOS VM to tweak configurations, use the process as described here:

```bash
# Mount the disk
mkdir -p /mnt/opencore && guestmount \
  -a /var/lib/libvirt/images/opencore.qcow2 \
  -m /dev/sda1 /mnt/opencore

# Go to partition
cd /mnt/opencore/EFI
```

Once inside the partition, you can make the modifications as desired. Here are some examples:

```bash
# EXAMPLE: update boot-args to add AMD GPU support
# NOTE: keepsyms is part of boot-args
sed -i 's/keepsyms=1/keepsyms=1 agdpmod=pikera/' OC/config.plist

# EXAMPLE: allow selecting a default entry with CTRL + Enter
# Search for AllowSetDefault: set to true
vim OC/config.plist

# EXAMPLE: reduce boot timeout
# Search for Timeout: set to 10
vim OC/config.plist
```

After finishing the necessary modifications, you must go outside of the partition and unmount the disk:

```bash
# Go to home location and unmount the disk
cd ${HOME}
guestunmount /mnt/opencore
```

----

Others tips and tricks:

- **[Linux Virtualization](6%20-%20Linux%20Virtualization.md)**
- **[Windows Virtualization](7%20-%20Windows%20Virtualization.md)**
