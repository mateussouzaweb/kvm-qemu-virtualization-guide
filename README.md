# KVM / QEMU Virtualization Guide

This guide is based on my own experience and hacks on running virtual machines. You already may have knowledge on this topic, so I will try to keep things simple and get directly to the point.

Our object is just one: have a **full working virtualization platform with QEMU using KVM as hypervisor** to run Linux, MacOS, Windows OS and any other operational system inside virtual machines with great performance, even for gaming! 

Finally, and most important, the guide is primarily designed to run Fedora Server as a hypervisor with [Cockpit](https://cockpit-project.org/) as graphical user interface but I am also including instructions to run on Fedora Workstation if you want a full desktop graphical interface experience on the hypervisor.

----

## What to Expect

- Easy guide if you already have KVM / QEMU knowledge.
- Virtualization that just works.
- Isolated and secure host system to just run VMs.
- Best practices for maximum performance in VMs.
- Linux, Windows and MacOS virtualization.
- Single GPU passthrough support (very easy to activate/deactivate).
- PCI passthrough ON DEMAND without need to pre-allocate devices on VFIO (or blacklist drivers on the OS).
- Audio and USB device passthrough.
- Live USB passthrough to supported VMs.
- CPU isolation and pinning support.
- Bridge network to keep every VMs available on the local network.
- Nested virtualization support for development environments.
- VM creation and management from CLI.
- Support for BOOT interface via CLI (to select the startup disk for example).
- More!

----

## How to Start

Follow the guides inside the ```Docs``` folder or use the links below for a specific topic:

- **[Host System Installation](Docs/0%20-%20Installation.md)**
- **[Configuring the Bridge Network](Docs/1%20-%20Bridge%20Network.md)**
- **[Enabling PCI-e Passthrough](Docs/2%20-%20PCI-e%20Passthrough.md)**
- **[Virtualization Hooks](Docs/3%20-%20Virtualization%20Hooks.md)**
- **[Managing Virtual Machines](Docs/4%20-%20Management.md)**
- **[XML Configurations](Docs/5%20-%20XML%20Configurations.md)**
  - **[Linux Virtualization](Docs/5.1%20-%20Linux.md)**
  - **[Windows Virtualization](Docs/5.2%20-%20Windows.md)**
  - **[MacOS Virtualization](Docs/5.3%20-%20MacOS.md)**
