# Linux Virtualization - Tips and Tricks

You can virtualize any Linux distribution and everything should work as expected without issues, just a few good observations...

## Observations

**Installation:**

- Installation runs fine with the GPU passthrough, you don't need to attach a virtual display to run installation.
- After installing, remember to install the ``qemu-guest-agent``.
- If you are using SPICE services, remember to install the  and ``spice-vdagent`` package.

**Graphics:**

- Since I have a 4k monitor, when I set the HiDPI scaling (2x) with both AMD and Nvidia GPUs, performance is very degraded making animation slow and the overall UI too big. I recently discovered that if I set the resolution to QHD (2560x1440), the system will run very smoothly, nice animations and good UI scaling. If you are having display problems like that, definitely try that!

**Sample XMLs:**

- You can check some configurations for Linux machines in the ``Samples/`` folder.
- Samples include examples for desktop for general purposes, including work.
- There are also a few examples of Linux server machines.

----

Others tips and tricks:

- **[Windows Virtualization](07%20-%20Windows%20Virtualization.md)**
- **[MacOS Virtualization](08%20-%20MacOS%20Virtualization.md)**
