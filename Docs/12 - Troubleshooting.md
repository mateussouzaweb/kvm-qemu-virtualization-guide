# Troubleshooting

If you are having issues running VMs in your environment, these general troubleshooting tips and tricks might help you.

## GPU Passthrough: VM Cannot Start or Breaks After Boot

When doing GPU passthrough, you may face a problem where the VM sometimes cannot start, or the VM starts but breaks immediately after the boot screen.

This problem affects recent AMD GPUs but may affect other vendors as well. The issue seems related to Secure Boot inside the VM: when the boot process needs to access advanced GPU features, the enrolled keys break it. To solve it, you have two options:

**Option 1** - Just try to run the VM again:

- Start the VM (on the first run it may freeze the system).
- Reboot the hypervisor system.
- Start the VM again (the enrolled keys should be there and the VM boots with no issues).
- Repeat the process for each new VM that you create.

**Option 2** - Disable Secure Boot on the VM by editing the XML specs:

```xml
<os firmware='efi'>
  <firmware>
    <feature enabled='no' name='enrolled-keys'/>
    <feature enabled='no' name='secure-boot'/>
  </firmware>
  <!-- remove <loader> and <nvram> entries -->
</os>
```

- Note that this option will not work on systems that require Secure Boot (Windows 11, for example).
- You need to remove the `<loader>` and `<nvram>` entries so they are created again based on the new firmware format.
- For more details, check the "**OS Optimizations**" section on [XML Configurations](Docs/05%20-%20XML%20Configurations.md).

## Vendor Reset for AMD GPUs

If you are using an AMD GPU, chances are that your device has the vendor-reset bug. This bug typically results in a complete black screen or even system freezes when you try to attach or detach the GPU on virtual machines. If you have a GPU from the series listed below, you need to install the vendor-reset package to fix this issue:

- **Polaris 10, 11 and 12**
- **Vega 10 and 20**
- **Navi 10, 12 and 14**

If your AMD GPU is not in that list, it most likely does not have the vendor-reset bug. To install the vendor-reset package on traditional systems, the steps are pretty simple: just run the following commands.

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora ONLY
# Update and install kernel packages
sudo dnf distro-sync
sudo dnf install -y dkms kernel-devel kernel-headers

# Install module
sudo git clone https://github.com/gnif/vendor-reset.git /usr/share/vendor-reset
cd /usr/share/vendor-reset
sudo dkms install .

# Load module on system boot
echo 'vendor-reset' | sudo tee /etc/modules-load.d/vendor-reset.conf

# Reboot hypervisor
sudo reboot
```

**NOTE:** I don't have a guide yet for vendor-reset on immutable systems. The steps above will work only on traditional systems.