# SteamOS Virtualization - Tips and Tricks

Did you know that you can virtualize SteamOS? 
The process is very easy and you can have the full Steam experience inside a Virtual Machine.  

## Observations

- Full gaming performance requires GPU passthrough. I don't recommend trying running the VM with virtual display.
- Special hardware rules are required, such as AMD GPU and correct disk placement.
- For some reason, boot screen is not visible, just wait a few seconds and you will see the SteamOS running.
- It works great!

## Installation

You need to download the Steam Recovery image from the [oficial source](https://help.steampowered.com/en/faqs/view/65B4-2AA3-5F37-4227). Follow the download process by accepting Steam terms and wait for the download conclusion. Once downloaded, the image will be on a `.bz2` file format and you need to extract it first, then move it to the correct location:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Extract file
bzcat steamdeck-repair-*.img.bz2 > steam-os.img
rm steamdeck-repair-*.img.bz2

# Move to target directory
sudo mv steam-os.img /var/lib/libvirt/images/steam-os.img
```

Now, download the sample XML configuration and tweak it to run on your system devices. This sample XML is very important because it reflect the requirements of SteamOS, such as disk on ``/dev/nvme0n1``:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Download sample
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Samples"
sudo curl -L ${REPOSITORY}/steam-os-passthrough.xml --output steam-os.xml

# Edit settings
sudo vim steam-os.xml
```

Finally define the VM and start it:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
virsh define steam-os.xml
virsh start steam-os
```

Once the system has booted you will see a KDE Plasma session with a few tools on the desktop. Use the "*Wipe Device & Install SteamOS*" option to perform Steam installation in the VM and reboot.

## Post Installation

After boot, the OS it will detect that it’s running inside a VM and will go straight to Desktop Mode.
Open the terminal and perform the actions as you like.

```bash
# Check current OS version
cat /etc/os-release

# Switch to beta branch and download latest version
steamos-select-branch beta
steamos-update check
steamos-update

# Enable developer mode, disabling read-only filesystem
steamos-devmode enable

# Switch to Wayland Desktop Mode
steamos-session-select plasma-wayland-persistent

# Switch to X11 Desktop Mode
steamos-session-select plasma-x11-persistent

# Switch to Gaming Mode
steamos-session-select
```

----

Others tips and tricks:

- **[Linux Virtualization](06%20-%20Linux%20Virtualization.md)**
- **[Windows Virtualization](07%20-%20Windows%20Virtualization.md)**
- **[MacOS Virtualization](08%20-%20MacOS%20Virtualization.md)**
