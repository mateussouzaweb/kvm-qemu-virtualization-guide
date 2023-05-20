# MacOS Virtualization - Tips and Tricks

MacOS has a few tricks to virtualize. Once running with their limitations, it works great and can be used as a daily machine.

## Observations

**Installation:**

- The installation process does not require virtualized display graphics if you have a compatible GPU.
- Otherwise, make sure you are using the virtualization display graphics.

**QEMU:**

- MacOS requires additional extra parameters to work. Consult how to use MacOS in KVM to learn more.
- Don't forget to include the QEMU XML namespace on top of the file to allow the usage of extra parameters.

**CPU:**

- If you have an AMD CPU, don't worry, it will work. However, nested virtualization on AMD CPUs is not possible (this will limit development environments in some cases).
- If your CPU is a Mac compatible Intel chip, change ``Haswell`` to ``host`` on the XML settings.

**Graphics:**

- GPU passthrough will not work on recent Nvidia graphic cards, forget it and buy a compatible AMD GPU.
- If you will passthrough any graphic card, just don't forget to remove the default video interface. Also make sure the gfx and audio devices are in the same bus (like ``0x01``) but in different function (``0x00`` and ``0x01``).
- Also, some AMD GPUs need to set ``agdpmod=pikera`` in the ``config.plist`` of OpenCore, at NVRAM boot-args section. See below a few options of how to edit OpenCore.

**Network:**

- Network interface model type must be ``vmxnet3``.
- Make sure the address is in bus ``0x0`` and slot ``0x0y`` (y is numeric).
- These settings will solve any issues that you can have on the installation process, built-in NIC, Apple Store and iCloud.

**Audio:**

- If you will passthrough onboard audio, make sure you put it in bus ``0x00`` and slot ``0x0y`` (y is numeric) as this is necessary to make AppleALC recognize its audio device.

**USB:**

- Live USB passthrough is not possible. You need to attach the USB device and then restart the VM to work.
- Attached USB keyboard at start may not work on OpenCore boot menu. To solve this, you must passthrough the USB Controller device.
- USB passthrough of USB 3.X devices may not work on MacOS.

**Boot:**

- If the keyboard does not work on OpenCore boot menu, you can use the Console to select the boot entry. Just enter the Console and press the arrow key to select the disk entry (no need to attach VNC graphics for it).
- If the default boot entry is on the recovery image, you can change it by selecting the MacOS volume and hitting ``CTRL`` + ``Enter`` to set this as the default boot entry. The change will be reflected in the next boot.

**OpenCore Settings:**

- This is just a list of few options to tweak that may be useful to you.
- Update ``boot-args`` to add AMD GPU support with the following extra text: ``agdpmod=pikera``.
- Update ``AllowSetDefault`` to allow selecting a default entry with ``CTRL`` + ``Enter``.
- Update ``Timeout`` to ``10`` in order to reduce boot timeout to 10s.
- Update ``Resolution`` to `Max` in order to enable max resolution.
- Update ``UIScale`` to ``2`` if you want to set support for HiDPI.

**Post Install:**

- Download the following apps to help tweak and modify OpenCore: OCAuxiliaryTools, Hackintool, ProperTree and GenSMBIOS.
- Remember to generate a unique SMBIOS and configure it in your EFI/OpenCore partition.
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
wget -O /var/lib/libvirt/images/fetch-macOS-v2.py ${REPOSITORY}/fetch-macOS-v2.py
wget -O /var/lib/libvirt/images/opencore.qcow2 ${REPOSITORY}/OpenCore/OpenCore.qcow2

# Fetch MacOS
chmod +x /var/lib/libvirt/images/fetch-macOS-v2.py
cd /var/lib/libvirt/images
./fetch-macOS-v2.py

# Convert DMG image to IMG format and remove unnecessary files
VERSION="ventura"
qemu-img convert BaseSystem.dmg -O raw macos-${VERSION}.img
rm -f BaseSystem.dmg BaseSystem.chunklist fetch-macOS-v2.py

# Fix permissions
chmod -R 660 /var/lib/libvirt/images/*
chown -R qemu:qemu /var/lib/libvirt/images
restorecon -R -vF /var/lib/libvirt/images
```

Now, download the sample XML configuration and tweak to run on your system devices:

```bash
# Download sample
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Samples"
curl -L ${REPOSITORY}/macos.xml --output macos.xml

# Edit settings
vim macos.xml
```

Here, you can also tweak the OpenCore partition before starting the VM (see the guide below). I will do it to enable the AMD GPU support for example, but if you don't know what to do for now, simply continue this guide...

Finally define the VM and start it:

```bash
virsh define macos.xml
virsh start macos
```

Done!

## Editing OpenCore EFI

If you need to mount the OpenCore partition outside the MacOS VM to tweak configurations, you can use another VM with Windows or Linux to access the partition in a GUI environment or use the process as described here to mount the EFI partition directly from the Hypervisor via terminal:

```bash
# Mount the disk
mkdir -p /mnt/opencore
guestmount -a /var/lib/libvirt/images/opencore.qcow2 -m /dev/sda1 /mnt/opencore

# Edit the file or do something else...
cd /mnt/opencore/EFI
vim OC/config.plist

# Unmount the disk once concluded modifications
# NOTE: You must go outside of the partition to allow unmount the disk
cd ${HOME}
guestunmount /mnt/opencore
```

This process is very useful if you are having issues booting MacOS.

----

Others tips and tricks:

- **[Linux Virtualization](6%20-%20Linux%20Virtualization.md)**
- **[Windows Virtualization](7%20-%20Windows%20Virtualization.md)**
