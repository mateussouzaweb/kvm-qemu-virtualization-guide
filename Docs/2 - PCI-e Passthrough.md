# Enabling PCI-e Passthrough

Follow the steps below to enable PCI-e passthrough and enable the redirection of hardware devices to the VMs.

First, make sure ``IOMMU`` and ``SVM`` (AMD) / ``VD-t`` (Intel) is enabled in your motherboard BIOS configuration. Reboot into your motherboard BIOS settings and check if these features are enabled.

Now, we will change the boot params of the host to enable the support for PCI-e Passthrough on the system:

```bash
vim /etc/sysconfig/grub
```

```bash
# Update this line by appending the options
GRUB_CMDLINE_LINUX="... $OPTIONS"
```

Replace ``$OPTIONS`` with the appropriated parameters for your case:

- Enable Intel IOMMU: ``intel_iommu=on iommu=pt``
- Enable AMD IOMMU: ``amd_iommu=on iommu=pt``
- Enable important features on AMD: ``kvm_amd.npt=1 kvm_amd.avic=1 kvm_amd.nested=1 kvm_amd.sev=1``
- Enable support for host CPU model inside VMs: ``kvm.ignore_msrs=1 kvm.report_ignored_msrs=0``
- Disable fallback video output: ``video=vesafb:off video=efifb:off``
- Make VFIO loads first: ``rd.driver.pre=vfio-pci``

Save the file once done. \
Now, enable the VFIO module on system load with the following command:

```bash
echo 'add_drivers+=" vfio vfio_iommu_type1 vfio_pci vfio_virqfd "' >> /etc/dracut.conf.d/vfio.conf
```

To finish, rebuild boot details and restart again:

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
dracut -f
reboot
```

After restart, check if everything is working as expected with the commands:

```bash
# VFIO settings
modprobe -c | grep vfio

# Libvirt check
virt-host-validate

# Single PCI-e check
lspci -vnn

# IOMMU groups check
for g in `find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V`; do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```

For IOMMU groups output, you should have your graphics card in a separate group. This will probably be ok with your main GPU, but for secondary GPUs (if your motherboard supports it), you may need to change the ACS implementation to allow more groups when your motherboard does not do that for you - the process is described below.

This is everything you need to allow PCI-e Passthrough. You now should be able to create VMs and attach PCI-e devices to it.

Next step: **[Setting up Virtualization Hooks](3%20-%20Virtualization%20Hooks.md)**

----

## ACS Patch to Fix IOMMU Groups

Some motherboards have a crappy IOMMU implementation, resulting in issues to passthrough PCI-e devices because they are attached to the same group as other devices. If you have such issue with your motherboard, fortunately there is a kernel patch (ACS override patch) that can solve these issues with a bit of security breach. You can install it following the commands below:

```bash
#
# For kernel 5.16.9
# Check other versions at https://github.com/some-natalie/fedora-acs-override/actions
#
LINK="https://github.com/some-natalie/fedora-acs-override/suites/5354626282/artifacts/167744633"

curl -o kernel-acs-override.zip -L ${LINK}
unzip kernel-acs-override.zip -d kernel-acs-override
cd kernel-acs-override
dnf install *.rpm
```

Edit the grub config and add the extra flag ``pcie_acs_override=downstream,multifunction`` on the parameters like you did before:

```bash
vim /etc/sysconfig/grub
```

To finish, rebuild boot details and restart:

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
dracut -f
reboot
```

Now, check the IOMMU groups again and see if that works. If everything is ok, your next step is: **[Setting up Virtualization Hooks](3%20-%20Virtualization%20Hooks.md)**
