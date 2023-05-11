# Enabling PCI-e Passthrough

PCI-e passthrough is a feature that allows the redirection of hardware devices to virtual machines. If you want to use your GPU in a VM for example, you need to enable such feature in the BIOS. Start by checking if the following BIOS settings are correctly configured in your motherboard:

- ``IOMMU`` - Enabled
- ``SVM`` or ``VD-t`` - Enabled
- ``ACS Override`` - Disabled
- ``Above 4G Decoding`` - Disabled
- ``Resizable BAR`` or ``Smart Access Memory`` - Disabled
- ``CSM Support`` - Disabled, must use UEFI

Consult the motherboard manual if you have doubts about how to find these settings, some of them may not be visible depending on your hardware. Once you have this correctly configured, follow the steps below to change the boot params of the host in order to enable the support for PCI-e Passthrough on the system:

```bash
vim /etc/default/grub
```

```bash
# Update this line by appending the below
GRUB_CMDLINE_LINUX="... $OPTIONS"
```

Replace ``$OPTIONS`` with the appropriate parameters for your case:

- **INTEL CPU:** Enables Intel IOMMU: ``intel_iommu=on iommu=pt``
- **AMD CPU:** Enables AMD IOMMU: ``amd_iommu=on iommu=pt``
- **AMD CPU:** Enables important features on AMD: ``kvm_amd.npt=1 kvm_amd.avic=1 kvm_amd.nested=1 kvm_amd.sev=1``
- **AMD CPU:** Enables support for host CPU model inside VMs: ``kvm.ignore_msrs=1 kvm.report_ignored_msrs=0``

The options below are dedicated to GPU passthrough:

- **ANY GPU:** Disables fallback video output: ``video=vesafb:off,efifb:off``
- **NVIDIA GPU:** Disables nouveau driver: ``nouveau.modeset=0``
- **AMD GPU:** Disables ASPM support: ``amdgpu.aspm=0``

Finally, the options below are related to drivers and PCI-e:

- **ANY:** Makes VFIO loads first: ``rd.driver.pre=vfio-pci``
- **ANY:** Enables extra PCI-e group separation: ``pcie_acs_override=downstream,multifunction``

Save the file once done.

Now, we need to enable the VFIO module on system load with the following command:

```bash
# Kernel 6.2+
echo 'add_drivers+=" vfio vfio_iommu_type1 vfio_pci "' >> /etc/dracut.conf.d/vfio.conf

# Kernel 6.1 or less
echo 'add_drivers+=" vfio vfio_iommu_type1 vfio_pci vfio_virqfd "' >> /etc/dracut.conf.d/vfio.conf
```

To finish, rebuild boot details and restart the system again:

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
dracut -f
reboot
```

After restart, check if everything is working as expected with the commands:

```bash
# Boot parameters
cat /proc/cmdline

# VFIO settings
modprobe -c | grep vfio

# Libvirt check
virt-host-validate | grep "QEMU"

# PCI-e IOMMU groups and USB bus check
# First download the helper command
REPOSITORY="https://mateussouzaweb.github.io/kvm-qemu-virtualization-guide/Scripts/bin"
curl -L ${REPOSITORY}/lspci-groups --output /usr/local/bin/lspci-groups
chmod +x /usr/local/bin/lspci-groups

# Now run it
lspci-groups

# You can also run it when need detailed information for a specific hardware type
lspci-groups | grep "USB"
lspci-groups | grep "VGA"
lspci-groups | grep "Audio"
lspci-groups | grep "Network"
```

For IOMMU groups output, you should have your graphics card in a separate group. This will probably be ok with your main GPU, but for secondary GPUs (if your motherboard supports it) or other devices, you may need to change the ACS implementation to allow more isolated groups when your motherboard does not do that for you - **if that is not your case and everything is ok, you can go to the next step**.

Go to <https://github.com/some-natalie/fedora-acs-override/actions> and download the latest kernel version available (you must log in on GitHub first to download files). After completing the download, if necessary, transfer the file to the host and run the following commands to install the patched kernel with alternative ACS implementation. You already added the boot flag ``pcie_acs_override`` so now just need to install the patched kernel:

```bash
# Install the new kernel
unzip kernel-*-acs-override-rpms.zip -d kernel-acs-override
dnf install --allowerasing kernel-acs-override/*.rpm

# Clean artifacts
rm -fr kernel-acs-override/ kernel-*-acs-override-rpms.zip

# Reboot
reboot
```

Check the IOMMU groups again to make sure it is working. You now should be able to create VMs and attach PCI-e devices to it.

Next step: **[Setting up Virtualization Hooks](3%20-%20Virtualization%20Hooks.md)**

----

## Vendor Reset for AMD GPUs

If you are using an AMD GPU, chances are that your device has the vendor reset bug. This bug always results in a complete black screen or even system freeze when you try to attach or detach the GPU on virtual machines. If you are lucky like me, you need to install the vendor reset package to fix this issue.

The steps are pretty simple, just run the following commands to install the package and fix the issue:

```bash
# Update and instal kernel packages
dnf distro-sync
dnf install -y dkms kernel-devel kernel-headers

# Install module
git clone https://github.com/gnif/vendor-reset.git /usr/share/vendor-reset;
cd /usr/share/vendor-reset
dkms install .

# Load module on system boot
echo 'vendor-reset' >> /etc/modules-load.d/vendor-reset.conf

# Reboot
reboot
```