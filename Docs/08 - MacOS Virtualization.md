# MacOS Virtualization - Tips and Tricks

MacOS has a few tricks to virtualize. Once running with their limitations, it works great and can be used as a daily machine.

## Observations

**Installation:**

- The installation process does not require virtualized display graphics if you have a compatible GPU.
- Otherwise, make sure you are using the virtualization display graphics.

**QEMU:**

- Virtual machines with MacOS requires additional extra parameters to work. Consult how to use MacOS in KVM to learn more.
- Don't forget to include the QEMU XML namespace on top of the file to allow the usage of extra parameters.

**CPU:**

- If you have an AMD CPU, don't worry, it will work. However, nested virtualization on AMD CPUs is not possible (this will limit development environments in some cases).
- If your CPU is a Mac compatible Intel chip, change ``Haswell-noTSX`` to ``host`` on the XML settings.

**Graphics:**

- If you will passthrough any graphic card, just don't forget to remove the default video interface. 
- Make sure the *gfx* and *audio* devices are in the same bus (like ``0x01``) but in different function (``0x00`` and ``0x01``) for GPU passthrough.
- GPU passthrough will not work on Nvidia graphic cards - forget it and buy a compatible AMD GPU.
- Some AMD GPUs need to set ``agdpmod=pikera`` in the ``config.plist`` of OpenCore, at "*NVRAM boot-args*" section - see below how to edit OpenCore from CLI if you need.
- Non-compatible RDNA 2 GPUs can be enable with [NootRx](https://github.com/ChefKissInc/NootRX). If you need this kext, do not append ``agdpmod=pikera`` in the ``config.plist`` and disable ``WhateverGreen`` kext.
- When doing GPU passthrough, boot can take a few minutes to finish - please be patient, it will work.

**Network:**

- Network interface model type must be ``vmxnet3`` because this is the only model compatible with MacOS.
- Make sure the address is in bus ``0x0`` and slot ``0x0y`` (y is numeric).
- These settings will solve any issues that you can have on the installation process, built-in NIC, Apple Store and iCloud.

**Audio:**

- If you will passthrough onboard audio, make sure you put it in bus ``0x00`` and slot ``0x0y`` (y is numeric) as this is necessary to make ``AppleALC`` recognize its audio device.

**USB:**

- Live USB passthrough is not possible. You need to attach the USB device and then restart the VM to work.
- Attached USB keyboard may not work on OpenCore boot menu.
- USB passthrough of USB 3.X devices may not work on MacOS.
- To avoid all of these issues, you should passthrough the entire USB Controller device to MacOS via PCI-e.

**Boot:**

- If the keyboard does not work on OpenCore boot menu, you can use the console connection to select the boot entry. Just enter the console with ``virsh console macos`` and press the arrow key to select the disk entry (no need to attach VNC graphics for it).
- If the default boot entry is on the recovery image, you can change it by selecting the MacOS volume and hitting ``CTRL`` + ``Enter`` to set this as the default boot entry. The change will be reflected in the next boot.

**OpenCore Settings:**

- This is just a list of few options to tweak that may be useful to you.
- Update ``boot-args`` to add AMD GPU support with the following extra text when necessary: ``agdpmod=pikera``.
- Update ``AllowSetDefault`` to allow selecting a default entry with ``CTRL`` + ``Enter``.
- Update ``Timeout`` to ``10`` in order to reduce boot timeout to 10s.
- Update ``Resolution`` to `Max` in order to enable max resolution.
- Update ``UIScale`` to ``2`` if you want to set support for HiDPI.

**Post Install:**

- Download the following apps to help tweak and modify OpenCore: [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools), [Hackintool](https://github.com/benbaker76/Hackintool), [ProperTree](https://github.com/corpnewt/ProperTree) and [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
- Remember to generate a unique SMBIOS and configure it in your EFI/OpenCore partition.
- Install additional kexts for your hardware if required - like [NootRx](https://github.com/ChefKissInc/NootRX) for example.

**Sample XMLs:**

- You can check some configurations for MacOS machines in the ``Samples/`` folder.
- Install instructions using this sample XML are available below.

## Installation

See: <https://github.com/kholia/OSX-KVM>

This guide uses the OSX-KVM project as base system. We grab just the necessary files to run it on QEMU. Start by downloading files and fetching the desired MacOS version:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Go to target directory
cd /var/lib/libvirt/images

# Download files
REPOSITORY="https://github.com/kholia/OSX-KVM/raw/master"
sudo wget -O fetch-macOS-v2.py ${REPOSITORY}/fetch-macOS-v2.py
sudo wget -O opencore.qcow2 ${REPOSITORY}/OpenCore/OpenCore.qcow2

# Fetch MacOS
sudo chmod +x fetch-macOS-v2.py
sudo ./fetch-macOS-v2.py

# Convert DMG image to IMG format and remove unnecessary files
VERSION="sequoia"
sudo qemu-img convert BaseSystem.dmg -O raw macos-${VERSION}.img
sudo rm -f BaseSystem.dmg BaseSystem.chunklist fetch-macOS-v2.py
```

Now, download the sample XML configuration and tweak to run on your system devices:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Download sample
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Samples"
sudo curl -L ${REPOSITORY}/macos.xml --output macos.xml

# Edit settings
sudo vim macos.xml
```

Here, you can also tweak the OpenCore partition before starting the VM (see the next section). I must do it to enable the AMD GPU support for my machine, but if you don't know what to do for now, simply continue this guide...

Finally define the VM and start it:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
virsh define macos.xml
virsh start macos
```

Done!

## Editing OpenCore EFI

If you need to mount the OpenCore partition outside the MacOS VM to tweak configurations, use the process as described here to mount the EFI partition directly from the Hypervisor via terminal:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Mount the disk
sudo mkdir -p /mnt/opencore
sudo guestmount -a /var/lib/libvirt/images/opencore.qcow2 -m /dev/sda1 /mnt/opencore

# Edit the file or do something else...
sudo vim /mnt/opencore/EFI/OC/config.plist

# Unmount the disk once concluded modifications
# NOTE: You must go outside of the partition to allow unmount the disk
cd ${HOME}
sudo guestunmount /mnt/opencore
```

You can also use another VM with Windows or Linux to access the partition in a GUI environment with [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools), [ProperTree](https://github.com/corpnewt/ProperTree) or another related program. 

This process is very useful if you are having issues booting MacOS.

----

Others tips and tricks:

- **[Linux Virtualization](06%20-%20Linux%20Virtualization.md)**
- **[Windows Virtualization](07%20-%20Windows%20Virtualization.md)**
