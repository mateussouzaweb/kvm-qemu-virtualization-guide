# KVM / QEMU Virtualization Guide

This guide is based on my own experience and hacks on running virtual machines. You already may have knowledge on this topic, so I will try to keep things simple and get directly to the point.

Our object is just one: have a **full working virtualization platform with QEMU using KVM as hypervisor** to run Linux, MacOS, Windows OS and any other operational system inside virtual machines with great performance, even for gaming! 

----

## What to Expect

- Virtualization solution with Fedora as hypervisor.
- Easy guide if you already have KVM / QEMU knowledge.
- Virtualization that just works.
- Up to date guide that explores the features of modern VFIO software.
- Best practices for maximum performance in VMs.
- Isolated and secure host system to just run VMs.
- Linux, Windows and MacOS virtualization.
- Single and Multi GPU passthrough support (very easy to activate/deactivate).
- PCI passthrough ON DEMAND without need to pre-allocate devices on VFIO (or blacklist drivers on the OS).
- Audio and USB device passthrough.
- Live USB passthrough on supported VMs.
- CPU isolation and pinning support.
- Bridge network to keep every VMs available on the local network.
- Nested virtualization for development environments.
- VM creation and management from CLI.
- Support for BOOT interface via CLI (to select the startup disk for example).
- More!

## My Current Setup

Just to provide additional context, here is my current hardware configuration:

- Motherboard: ASRock X570 Phantom Gaming 4
- Memory: Corsair Vengeance 32GB 2666MHz DDR4
- CPU: AMD Ryzen 5700X
- GPU: AMD RX 6700XT XFX QICK 319 12GB
- SSD: Crucial SSD P2 2TB NVMe PCIe M.2

This guide also was previously tested with the following hardware:

- Motherboard: Gigabyte B450 Aorus Pro Wifi
- Memory: Corsair Vengeance 16GB 2133MHz DDR4
- CPU: AMD Ryzen 1700X
- GPU: AMD RX 5500XT Gigabyte Gaming OC 4G
- GPU: Nvidia GTX 1070 Gigabyte G1 Gaming 8GB

## How to Start

Follow the guides inside the ```Docs``` folder or use the links below for a specific topic:

- **[Host System Installation](Docs/00%20-%20Installation.md)**
- **[Configuring the Bridge Network](Docs/01%20-%20Bridge%20Network.md)**
- **[Enabling PCI-e Passthrough](Docs/02%20-%20PCI-e%20Passthrough.md)**
- **[Virtualization Hooks](Docs/03%20-%20Virtualization%20Hooks.md)**
- **[Managing Virtual Machines](Docs/04%20-%20Virtual%20Machine%20Management.md)**
- **[XML Configurations](Docs/05%20-%20XML%20Configurations.md)**
- **[Linux Virtualization](Docs/06%20-%20Linux%20Virtualization.md)**
- **[Windows Virtualization](Docs/07%20-%20Windows%20Virtualization.md)**
- **[MacOS Virtualization](Docs/08%20-%20MacOS%20Virtualization.md)**
- **[File Sharing](Docs/09%20-%20File%20Sharing.md)**
- **[Docker Containers](Docs/10%20-%20Docker%20Containers.md)**
