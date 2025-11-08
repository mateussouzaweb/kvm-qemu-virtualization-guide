# Enabling PCI-e Passthrough

PCI-e passthrough is a feature that allows the redirection of hardware devices to virtual machines. If you want to use your GPU in a VM for example, you need to enable such feature in the BIOS. Start by checking if the following BIOS settings are correctly configured in your motherboard:

- ``IOMMU`` - Enabled
- ``SVM`` or ``VD-t`` - Enabled
- ``SR-IOV Support`` - Enabled
- ``ACS Enable`` or ``ACS Override`` - Auto or Disabled
- ``Above 4G Decoding`` - Enabled or Disabled
- ``Resizable BAR`` or ``Smart Access Memory`` - Disabled
- ``PCIe ARI Support`` - Auto or Enabled
- ``Enable AER Cap`` - Auto or Enabled
- ``CSM Support`` - Disabled, must use UEFI

Consult the motherboard manual if you have doubts about how to find these settings, but be aware that some of them may not be visible depending on your hardware.

## Knowing Boot Parameters

Boot parameters are responsible to set additional configurations in the system when kernel is initiating and should be used in conjunction with the BIOS settings. Any Linux distribution can be tweaked with boot parameters but they must match your current hardware specs to properly enable PCI-e passthrough.

We will list the most important options here and you should take note for the most appropriate parameters for your case:

- **INTEL CPU:** Enables Intel IOMMU: ``intel_iommu=on iommu=pt``
- **AMD CPU:** Enables AMD IOMMU: ``amd_iommu=on iommu=pt``
- **AMD CPU:** Enables important features on AMD: ``kvm_amd.npt=1 kvm_amd.avic=1 kvm_amd.nested=1 kvm_amd.sev=1``

The options below are useful when your CPU is not able to natively support nested virtualization:

- **ANY CPU:** Enables support for host CPU model inside VMs: ``kvm.ignore_msrs=1``
- **ANY CPU:** Ignore CPU model errors VMs: ``kvm.report_ignored_msrs=0``

The options below are dedicated to GPU passthrough:

- **ANY GPU:** Disables fallback video output: ``video=vesafb:off,efifb:off,simplefb:off``
- **NVIDIA GPU:** Increase compatibility of nouveau with VFIO: ``nouveau.modeset=0``
- **NVIDIA GPU:** Completely disable nouveau driver: ``modprobe.blacklist=nouveau rd.driver.blacklist=nouveau``
- **AMD GPU:** Disables ASPM support: ``amdgpu.aspm=0``

Finally, the options below are related to drivers and PCI-e:

- **ANY:** Makes VFIO loads first on Linux boot: ``rd.driver.pre=vfio-pci``
- **ANY:** Enables extra PCI-e group separation on patched kernel implementation: ``pcie_acs_override=downstream,multifunction``

If you want to make any specific PCI-e device like a network card or a secondary GPU to be exclusively eligible and controlled by VFIO, you can also declare such devices in the boot parameters. Use the command ``lspci -nn`` to get the vendor and device code of the device (for example: ``15b3:1003``) and declare it with the format below:

- **ANY:** Declarares PCI-e devices managed by VFIO: ``vfio_pci.ids=${DEVICE},...``

Please take note for the most appropriate parameters for your case, they will be necessary for the next topic.

## Adding Boot Parameters

Now that you know the available boot parameters, follow the steps below to enable the support for PCI-e passthrough on traditional systems:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora ONLY
# Enable VFIO modules on system load
sudo sh -c "echo 'add_drivers+=\" vfio vfio_iommu_type1 vfio_pci \"' > /etc/dracut.conf.d/vfio.conf"

# Edit GRUB options
# Search for the line containing ``GRUB_CMDLINE_LINUX``
# Update this line by appending the appropriate parameters for your case
# GRUB_CMDLINE_LINUX="... $PARAMETERS"
# Save the file once done
sudo vim /etc/default/grub

# After tweak rebuild boot details
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
sudo dracut -f

# Reboot hypervisor
sudo reboot
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu ONLY
# Enable VFIO modules on system load
sudo sh -c "echo 'add_drivers+=\" vfio vfio_iommu_type1 vfio_pci \"' > /etc/dracut.conf.d/vfio.conf"

# Edit GRUB options
# Search for the line containing ``GRUB_CMDLINE_LINUX``
# Update this line by appending the appropriate parameters for your case
# GRUB_CMDLINE_LINUX="... $PARAMETERS"
# Save the file once done
sudo vim /etc/default/grub

# After tweak rebuild boot details
sudo update-grub
sudo dracut -f

# Reboot hypervisor
sudo reboot
```

If you are running an immutable system, the process is different and you should append every boot parameter in OSTree on ``kargs`` with the ``--append-if-missing="$PARAMETER"`` option. I will provide a two samples here, but you should tweak it based on our needs:

![Fedora](../Images/fedora.png)
**FEDORA - ATOMIC:**

```bash
# Fedora Immutable ONLY
# Enable VFIO modules on system load
rpm-ostree initramfs --enable --arg="--add-drivers" --arg="vfio-pci"

# AMD CPUs ONLY
# Append boot parameters
rpm-ostree kargs \
    --append-if-missing="amd_iommu=on" \
    --append-if-missing="iommu=pt" \
    --append-if-missing="kvm_amd.npt=1" \
    --append-if-missing="kvm_amd.avic=1" \
    --append-if-missing="kvm_amd.nested=1" \
    --append-if-missing="kvm_amd.sev=1" \
    --append-if-missing="video=vesafb:off,efifb:off,simplefb:off" \
    --append-if-missing="rd.driver.pre=vfio_pci"

# Intel CPUs ONLY
# Append boot parameters
rpm-ostree kargs \
    --append-if-missing="intel_iommu=on" \
    --append-if-missing="iommu=pt" \
    --append-if-missing="video=vesafb:off,efifb:off,simplefb:off" \
    --append-if-missing="rd.driver.pre=vfio_pci"

# Reboot hypervisor
sudo reboot
```

After restart, you can check if PCI-e passthrough is working with the instructions on the next topic.

## Checking Passthrough Support

You can check if everything is working as expected with the below commands:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Boot parameters
cat /proc/cmdline

# VFIO settings
modprobe -c | grep vfio

# Libvirt check
virt-host-validate | grep "QEMU"

# Dmesg check
sudo dmesg | grep -i -e DMAR -e IOMMU

# PCI-e IOMMU groups and USB bus check
# First download the helper command
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Scripts/bin"
sudo curl -L ${REPOSITORY}/lspci-groups --output /usr/local/bin/lspci-groups
sudo chmod +x /usr/local/bin/lspci-groups

# Now run it
lspci-groups

# You can also run it when need detailed information for a specific hardware type
lspci-groups | grep "USB"
lspci-groups | grep "VGA"
lspci-groups | grep "Audio"
lspci-groups | grep "Network"
lspci-groups | grep "NVMe"
lspci-groups | grep "SSD"
```

For IOMMU groups output, you should have your graphics card in a separate group. This will probably be ok with your main GPU, but for secondary GPUs (if your motherboard supports it) or other devices, you may need to change the ACS implementation to allow more isolated groups when your motherboard does not do that for you - **if that is not your case and everything is ok, you can go to the next step**, otherwise, follow the next topic to override ACS implementation.

## Overriding ACS Implementation

**NOTE:** I don't have a guide yet for overriding ACS implementation on Fedora immutable systems, the steps below will work only on Fedora traditional systems.

If you are running Fedora system and need to separate IOMMU groups, go to <https://github.com/some-natalie/fedora-acs-override/actions> and download the latest kernel version available (you must log in on GitHub first to download files). After completing the download, if necessary, transfer the file to the host and run the following commands to install the patched kernel with alternative ACS implementation. You probably already added the boot flag ``pcie_acs_override`` so now just need to install the patched kernel:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora ONLY
# Install the new kernel
sudo unzip kernel-*-acs-override-rpms.zip -d kernel-acs-override
sudo dnf install --allowerasing kernel-acs-override/*.rpm

# Clean artifacts
sudo rm -fr kernel-acs-override/ kernel-*-acs-override-rpms.zip

# Reboot hypervisor
sudo reboot
```

Check the IOMMU groups again to make sure it is working. You now should be able to create VMs and attach PCI-e devices to it.

## Vendor Reset for AMD GPUs

**NOTE:** I don't have a guide yet for vendor reset on immutable systems, the steps below will work only on traditional systems.

If you are using an AMD GPU, chances are that your device has the vendor reset bug. This bug will always result in a complete black screen or even system freezes when you try to attach or detach the GPU on virtual machines. If you are lucky like me and have a GPU from the series listed below, you need to install the vendor reset package to fix this issue:

- **Polaris 10, 11 and 12**
- **Vega 10 and 20**
- **Navi 10, 12 and 14**

If your AMD GPU is not in that list, then your GPU definitely does not have the vendor reset bug. To install the vendor reset package in traditional systems, the steps are pretty simple, just run the following commands to install the package and fix the issue:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora ONLY
# Update and instal kernel packages
sudo dnf distro-sync
sudo dnf install -y dkms kernel-devel kernel-headers

# Install module
sudo git clone https://github.com/gnif/vendor-reset.git /usr/share/vendor-reset;
cd /usr/share/vendor-reset
sudo dkms install .

# Load module on system boot
sudo echo 'vendor-reset' > /etc/modules-load.d/vendor-reset.conf

# Reboot hypervisor
sudo reboot
```

----

Next step: **[Setting up Virtualization Hooks](03%20-%20Virtualization%20Hooks.md)**