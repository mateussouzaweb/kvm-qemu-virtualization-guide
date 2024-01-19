# Windows Virtualization - Tips and Tricks

You can virtualize almost any Windows version. These observations are valid for Windows 10 and Windows 11. Besides these few observations, everything else should work as expected.

## Observations

**Installation:**

- For Windows 11 installations, make sure that Secure Boot and TPM 2.0 are configured in the virtual machine.
- Installation process will require a virtualized display graphics.
- In the disk selection process, you must install the VirtIO driver if disks are not visible. Make sure you have the VirtIO ISO configured as a CDROM device.
- After installing, remember to install VirtIO software.

**Graphics:**

- Use the patched VGA BIOS on Nvidia GPUs.
- Just install the GPU driver of the manufacturer (AMD/Nvidia) and GPU should work.
- After the driver has been installed, remove the virtualized display graphics if you are having issues.

**USB:**

- Live USB passthrough does not work on Windows before login. You need to start Windows with an attached USB mouse/keyboard device and then after logging in, every USB device should work as expected.

**Network:**

- Network interface model type must be ``e1000e`` if you want to have internet in the OS install process.
- If you are having issues connecting to the network, it is recommended change the model type from ``virtio`` to ``e1000e``, but ``virtio`` will always have more performance. 
- If you are having issues streaming the VM to any other device, this maybe is not a network issue. Try disable the GPU Scheduler on "Settings" > "Display" > "Graphics" > "Change default graphics settings" first and see if that solves the problem.

**Sample XMLs:**

- You can check some configurations for Windows machines in the ``Samples/`` folder.
- These settings are very good for gaming, including gaming with VR headsets.

----

Others tips and tricks:

- **[Linux Virtualization](06%20-%20Linux%20Virtualization.md)**
- **[MacOS Virtualization](08%20-%20MacOS%20Virtualization.md)**
