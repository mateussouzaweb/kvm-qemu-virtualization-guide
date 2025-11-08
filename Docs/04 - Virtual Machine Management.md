# Managing Virtual Machines

This section will show you some commands necessary to manage VMs from the terminal. You will also learn how to manage GPU BIOS, ISOs, Disks, other hardware parts dependencies and permissions to finally know how to create and configure the first virtual machine.

## Commands to Manage VMs

These are some basic commands to manage VMs:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
virsh list --all # List all VMs
virsh define ${NAME}.xml # Define VM from XML file
virsh edit ${NAME} # Edit VM XML
virsh start ${NAME} # Start the VM
virsh console ${NAME} # Enters in VM console
virsh reboot ${NAME} # Reboot the VM
virsh shutdown ${NAME} # Shutdown the VM
virsh destroy ${NAME} # Force shutdown on the VM
virsh suspend ${NAME} # Suspend the VM
virsh resume ${NAME} # Resume the VM after suspend
```

If you want enable or disable autostart on VM when you power up the host machine, run the following command:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Enable
virsh autostart ${NAME}

# Disable
virsh autostart --disable ${NAME}
```

If you need to manage multiple VMs at once, here are a few helpful commands:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Stop all running VMs
for i in `virsh list | grep running | awk '{print $2}'`; do
  virsh shutdown $i
done

# Edit all available VMs
for i in `virsh list --all --name`; do
  virsh edit $i
done

# Define multiple VMs from XML specs
for XML in ./*.xml; do
  virsh define $XML
done
```

## VGA BIOS Management

To avoid issues with GPU cards inside VMs, we recommend extracting the GPU ROM BIOS. If you know that your GPU card doesn't need it, just skip this process, otherwise, you need to run this just once.

Use GPU-z on Windows standalone machine to extract the ROM file for the GPU or download the BIOS based on your hardware at <https://www.techpowerup.com/vgabios/>. To extract the BIOS file from Linux, you must run a live USB distro with CLI mode, then run the following command:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Ensure to run as sudo
sudo su

# Perform BIOS extraction
echo 1 > /sys/bus/pci/devices/{$ID}/rom
cat /sys/bus/pci/devices/{$ID}/rom > ${NAME}.rom
echo 0 > /sys/bus/pci/devices/{$ID}/rom
```

After extracting, login on the main host and run the command below to copy the ROM to the correct location:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# IMPORTANT: Every GPU BIOS file must be located at /var/lib/libvirt/vbios/
# Create folder if not exists
sudo mkdir -p /var/lib/libvirt/vbios/

# Go to target directory
cd /var/lib/libvirt/vbios/

# Copy ROM file from where you saved it
sudo cp /path/of/NAVI48.rom NAVI48.rom
```

**NVIDIA only**: If you need to patch the GPU BIOS in order to avoid ``code 43`` issue, use the following command to create a patched ROM version of your GPU BIOS:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Ensure to run as sudo
sudo su

# Perform BIOS modification
echo "`grep -aoib video ${NAME}.rom | \
  cut -d: -f1` / 16 * 16" | bc | \
  xargs -I OFF dd if=${NAME}.rom of=${NAME}-P.rom bs=OFF skip=1
```

## ISO Management

To install operational systems inside VMs, you probably need an ISO file. You can copy the ISOs from another hard drive or download the ISO file to the correct location as show below:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# IMPORTANT: Every ISO must be located at /var/lib/libvirt/images/
# Create folder if not exists
sudo mkdir -p /var/lib/libvirt/images/

# Go to target directory
cd /var/lib/libvirt/images/

# Copy ISO file from where you saved it
sudo cp /path/of/ubuntu.iso ubuntu.iso

# Download the ISO that you want
sudo wget -O ubuntu-desktop-24.04.iso https://releases.ubuntu.com/noble/ubuntu-24.04.3-desktop-amd64.iso

# Download virtio drivers for Windows
sudo wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso
```

## Disk Management

Virtual machines likely require a disk to install and run their software. Here, you have a few options like directly attaching a physical disk, creating a LVM partition or creating a virtual disk for this propose with ``.qcow2`` or ``.raw`` disk images format. Use one of the commands below to create the desired disk format for the VM:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Go to target directory
cd /var/lib/libvirt/images/

# LVM partition
sudo lvcreate -n ${NAME} -L 100G ${VOLUME_GROUP}

# Raw disk
sudo qemu-img create -f raw ${NAME}.raw 100G

# QCOW2 disk
sudo qemu-img create -f qcow2 ${NAME}.qcow2 100G
```

When necessary, you can also convert between formats or transform your physical disk to virtual:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Convert qcow2 to raw
sudo qemu-img convert -p -O raw ${NAME}.qcow2 ${NAME}.raw

# Convert physical to virtual
sudo qemu-img convert -p -O raw /dev/nvme1n1 ${NAME}.raw
```

You don't need to format the disk or partition, just pass it to the VM and run the installation process.

## Storage Pools Management

Storage pools are a way to organize and manage storage for virtual machines (VMs) on a host that offers great flexibility and easier maintenance. They include local directories with virtual disks, directly attached disks, physical partitions, and logical volume management (LVM)  groups on local devices.

By default, KVM/QEMU includes a storage pool named as ``default`` that also points to the default directory where your virtual Disks and ISOs should be located: ``/var/lib/libvirt/images/``. We recommend using this storage pool when possible.

We will provide just the basic commands here for the ``default`` storage pool:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Start storage pool
virsh pool-start default

# Turn on autostart
virsh pool-autostart default

# Refresh pool when adding new disks
virsh pool-refresh default
virsh pool-refresh disks
```

Consult the oficial manual to know how to handle storage pools with libvirt if you wan't to create others storage pools or move the default storage pool location for more advanced cases.

## Permission Management

Security are a important aspect of virtual machines and Linux in general. Before trying to running virtual machines, you should make sure that the file permissions are properly defined when using a operating system with SELinux (Fedora) or AppArmor (Ubuntu).

The process is very easier and you can follow the same steps to set the correct permissions on VGA BIOS, in virtual disks, ISO files and others:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER / ATOMIC:**

```bash
# Fedora ONLY
# Go to target folder
cd /var/lib/libvirt/images/

# Fix permissions on current folder files
sudo chmod +rwx ${PWD}
sudo chmod -R 660 ${PWD}/*
sudo chown -R qemu:qemu ${PWD}
sudo restorecon -R -vF ${PWD}
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu ONLY
# Go to target folder
cd /var/lib/libvirt/images/

# Fix permissions on current folder files
sudo chown libvirt-qemu:kvm ${PWD}/*
sudo chmod 660 ${PWD}/*
sudo systemctl reload apparmor
sudo systemctl restart libvirtd
```

Make sure to run these permissions fixes after manipulating files used to run virtual machines.

## Creating a Virtual Machine from CLI

You can create and configure a new virtual machine directly from the CLI. First, you need to know the list of OS variants available to install and the hardware parts that can be attached to the VM:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
osinfo-query os # OS list
virsh nodedev-list # PCI-E parts
lsusb # USB devices
lsblk # Disks
```

Isolate the relevant info and save such information to use later on the VM configuration.

To create a VM directly from CLI, use the commands as below. This process is slightly different from the method available everywhere because we will first generate one sample XML based on the described OS, then we will tweak the settings and finally create a new VM based on the specs that we declared:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Edit variables
CDROM="/var/lib/libvirt/images/ubuntu-desktop-25.04.iso"
DISK="/dev/hypervisor/ubuntu"
VARIANT="ubuntu25.04"
NAME="ubuntu"

# Generate basic XML config
# 4 core / 8 threads CPU
# 8GB ram
# Bridge network
# VNC graphics
virt-install \
  --print-xml \
  --import \
  --hvm \
  --boot uefi \
  --cpu host-passthrough,+topoext \
  --cpu topology.cores=4,topology.sockets=1,topology.threads=2 \
  --cpu cache.mode=passthrough \
  --vcpus 8,cpuset=8-15 \
  --memory 8192 \
  --network bridge=virbr0 \
  --graphics vnc,listen=0.0.0.0 \
  --features kvm.hidden.state=on \
  --features vmport.state=off \
  --serial type=pty,target.model.name=isa-serial,target.port=0,target.type=isa-serial \
  --console type=pty,target.type=serial,target.port=0 \
  --channel unix,target_type=virtio,name=org.qemu.guest_agent.0 \
  --disk format=raw,cache=writeback,io=io_uring,discard=unmap,path=${DISK} \
  --disk device=cdrom,bus=sata,source.file=${CDROM} \
  --os-variant ${VARIANT} \
  --name ${NAME} \
  > ${NAME}.xml
```

Now that you generated the basic XML, you need to edit the XML file to set extra configurations and tweak your VM configuration. The best configurations are described on the next topic - **[XML Configurations](05%20-%20XML%20Configurations.md)**:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
vim ${NAME}.xml
```

After making the necessary changes on the XML configuration, import the XML file to create the virtual machine and you can start the VM:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
virsh define ${NAME}.xml
virsh start ${NAME}
virsh console ${NAME}
```

That is all. Install the operating system and be happy!