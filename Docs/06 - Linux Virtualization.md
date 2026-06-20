# Linux Virtualization - Tips and Tricks

You can virtualize any Linux distribution and everything should work as expected without issues, just a few good observations...

## Observations

**Installation:**

- Installation runs fine with the GPU passthrough, you don't need to attach a virtual display to run installation.
- After installing, remember to install the ``qemu-guest-agent``.
- If you are using SPICE services, remember to install the  and ``spice-vdagent`` package.
- If you host system is installed with ``btrfs`` avoid using the same filesystem inside the virtual machine since it will give no benefit to you and you will get an performance hit on these Virtual Machines simply by choosing the wrong filesystem.

**Sample XMLs:**

- You can check some configurations for Linux machines in the ``Samples/`` folder.
- Samples include examples for desktop for general purposes, including work.
- There are also a few examples of Linux server machines.

----

Others tips and tricks:

- **[Windows Virtualization](07%20-%20Windows%20Virtualization.md)**
- **[MacOS Virtualization](08%20-%20MacOS%20Virtualization.md)**
- **[SteamOS Virtualization](09%20-%20SteamOS%20Virtualization.md)**
