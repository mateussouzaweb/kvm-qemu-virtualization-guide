# Windows Virtualization - Tips and Tricks

You can virtualize almost any Windows version. These observations are valid for Windows 10 and Windows 11. Besides these few observations, everything else should work as expected.

## Observations

**Installation:**

- For Windows 11 installations, make sure that Secure Boot and TPM 2.0 are configured in the virtual machine.
- Installation runs fine with the GPU passthrough, you don't need to attach a virtual display to run installation.
- The correct network interface model is also important for internet access. See the notes below for more details.
- Make sure you have the VirtIO ISO configured as a CDROM device during the installation process.
- Once you are in the disk selection process of Windows installation, you must install the VirtIO drivers from the CDROM if disks are not visible.
- After installing, remember to install VirtIO software from the VirtIO ISO. This software includes drivers for QEMU Guest Agent and SPICE guest agent and drivers.

**Graphics:**

- Remember to use the patched VGA BIOS on Nvidia GPUs.
- Just install the GPU driver of the manufacturer (AMD/Nvidia) and GPU should work.
- After the driver has been installed, remove the virtualized display graphics if you are having display issues.

**USB:**

- Live USB passthrough does not work on Windows before login. You need to start Windows with an attached USB mouse/keyboard device and then after logging in, every USB device should work as expected.

**Network:**

- For Windows, there are basically 3 network interfaces models on Qemu: ``virtio``, ``igb`` and ``e1000e``.
- You must set the network interface model as  ``igb`` or ``e1000e`` to have internet access during the Windows installation process.
- Once Windows has been installed, install the VirtIO drivers and set the network as ``virtio`` because this model will always have more performance than others - supports 10G networks for example.
- If you are having issues streaming the VM to any other devices, this maybe is not a network issue. Try disable the GPU Scheduler on "Settings" > "Display" > "Graphics" > "Change default graphics settings" first and see if that solves the problem.

**Sample XMLs:**

- You can check some configurations for Windows machines in the ``Samples/`` folder.
- These settings are very good for gaming, including gaming with VR headsets.

----

Others tips and tricks:

- **[Linux Virtualization](06%20-%20Linux%20Virtualization.md)**
- **[MacOS Virtualization](08%20-%20MacOS%20Virtualization.md)**
