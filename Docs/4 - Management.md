# Managing Virtual Machines

Before creating and configuring the first virtual machine, it's important to learn how to manage GPU BIOS, ISOs, Disks and hardware parts dependencies. These topics are described here.

## VGA BIOS Management

**INFO:** Every GPU BIOS must be located at ``/usr/share/vgabios/``.

To avoid issues with GPU cards inside VMs, we recommend extracting the GPU ROM BIOS. If you know that your GPU card doesn't need it, just skip this process, otherwise, you need to run this just once.

Use GPU-z on Windows standalone machine to extract the ROM file for the GPU or download the BIOS at <https://www.techpowerup.com/vgabios/>. To extract the BIOS file from Linux, you must run a live USB distro with CLI mode, then run the following command:

```bash
echo 1 > /sys/bus/pci/devices/{$ID}/rom
cat /sys/bus/pci/devices/{$ID}/rom > ${NAME}.rom
echo 0 > /sys/bus/pci/devices/{$ID}/rom
```

After extracting, login on the main host and run the command below to copy the ROM to the correct location:

```bash
# Create folder if not exists
mkdir -p /usr/share/vgabios/

# Copy ROM file from where you saved it
cd /mnt/files/
cp GP104.rom /usr/share/vgabios/ # Nvidia GPU
cp NAVI14.rom /usr/share/vgabios/ # AMD GPU
```

**NVIDIA only**: If you need to patch the GPU BIOS in order to avoid ``code 43`` issue, use the following command to create a patched ROM version of our GPU BIOS:

```bash
echo "`grep -aoib video ${NAME}.rom | \
  cut -d: -f1` / 16 * 16" | bc | \
  xargs -I OFF dd if=${NAME}.rom of=${NAME}-P.rom bs=OFF skip=1
```

That is all, just remember the name of the ROM file when creating or configuring the GPU device into the virtual machine.

## ISO Management

**INFO:** Every ISO must be located at ``/var/lib/libvirt/images/``.

To install operational systems inside VMs, you probably need an ISO. Since we are using a system that does not have graphical interface, you can download OS ISOs from the terminal. Use the steps below to do it:

```bash
# Create if not exists and go to folder
mkdir -p /var/lib/libvirt/images/
cd /var/lib/libvirt/images/

# Now download the ISO you want
wget https://releases.ubuntu.com/20.04/ubuntu-20.04.3-desktop-amd64.iso

# VirtIO drivers for Windows
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso
```

You can also copy the ISOs from another hard drive if necessary. Mount the drive and use the following command to copy the file to the correct location:

```bash
# Copy file
cp ubuntu.iso /var/lib/libvirt/images/ubuntu.iso
```

Just as a reminder, the installation of Fedora Server by default uses only a small portion of your disk in the LVM mode which is good because in your setup, we will allocate LVM partitions to each VM. If the root disk does not have the size that you need to store ISOs or something else, you can resize the disk with the command like below:

```bash
# Add 100GB to the root disk
lvextend -r -L +100G /dev/${VOLUME_GROUP}/root
```

Once creating the VM, just remember the name of the ISO file.

## Disk Management

Every virtual machine will need an exclusive disk to run. Usually, ``.qcow2`` or ``.raw`` disk images are used on this subject, but we will use a different approach with directly attached disk or LVM partitions targeting best performance. If you don't have an extra high-speed disk like SSD or NVME disk to directly attach to the VM, use the command below to create a new partition exclusively to that VM inside the host disk:

```bash
lvcreate -n ${NAME} -L 100G ${VOLUME_GROUP}
```

You don't need to format the partition, just pass it to the VM and run the installation process.

## Creating a Virtual Machine from CLI

Now that you have solved the GPU BIOS, ISO and Disk dependencies for the VM, you can create and configure a new virtual machine. We just need to know the list of OS variants available to install and the hardware parts that can be attached to the VM:

```bash
osinfo-query os # OS list
virsh nodedev-list # PCI-E parts
lsusb # USB devices
lsblk # Disks
```

Isolate the relevant info and save such information to use later on the VM configuration.

To create a VM directly from CLI, use the following command. This process is slightly different from the process using the default method available everywhere because we will first generate one sample XML based on the described OS, tweak its settings and finally create a new VM based on the specs that we declared:

```bash
# Edit variables
CDROM="ubuntu-21.10.iso"
DISK="/dev/hypervisor/ubuntu"
VARIANT="ubuntu21.10"
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
  --network bridge=br0 \
  --graphics vnc,listen=0.0.0.0 \
  --features kvm.hidden.state=on \
  --features vmport.state=off \
  --serial type=pty,target.model.name=isa-serial,target.port=0,target.type=isa-serial \
  --console type=pty,target.type=serial,target.port=0 \
  --channel unix,target_type=virtio,name=org.qemu.guest_agent.0 \
  --disk format=raw,cache=writeback,io=io_uring,discard=unmap,path=${DISK} \
  --disk device=cdrom,bus=sata,source.file=/var/lib/libvirt/images/${CDROM} \
  --os-variant ${VARIANT} \
  --name ${NAME} \
  > ${NAME}.xml
```

Now that you generated the basic XML, you need to edit the XML file to set extra configurations and tweak your VM configuration - the configurations will be described on the [next topic](5%20-%20XML%20Configurations.md), we are just describing the process of creating a new virtual machine here:

```bash
vim ${NAME}.xml
```

After making the necessary changes on the XML configuration, import the XML file to create the virtual machine and you can start the VM:

```bash
virsh define ${NAME}.xml
virsh start ${NAME}
virsh console ${NAME}
```

That is all. Install the operating system and be happy! \
See next topic to learn how to properly configure you XML file using hooks: **[XML Configurations](5%20-%20XML%20Configurations.md)**.

----

## Commands to Manage VMs

Basic commands:

```bash
virsh list --all # List all VMs
virsh edit ${NAME} # Edit VM XML
virsh start ${NAME} # Start the VM
virsh console ${NAME} # Enters in VM console
virsh reboot ${NAME} # Reboot the VM
virsh shutdown ${NAME} # Shutdown the VM
virsh destroy ${NAME} # Force shutdown on the VM
virsh suspend ${NAME} # Suspend the VM
virsh resume ${NAME} # Resume the VM after suspend
```

If you want enable or disable autostart on VM when you power up the host server, run the following command:

```bash
# Enable
virsh autostart ${NAME}

# Disable
virsh autostart --disable ${NAME}
```

To see the current hooks parameters for the VM:

```bash
virsh desc ${NAME}
```

To stop all running VMs:

```bash
for i in `virsh list | grep running | awk '{print $2}'`; do
  virsh shutdown $i
done
```

To edit all available VMs:

```bash
for i in `virsh list --all --name`; do
  virsh edit $i
done
```
