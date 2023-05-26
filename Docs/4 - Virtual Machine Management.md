# Managing Virtual Machines

This section will show you some commands necessary to manage VMs from the terminal. You will also learn how to manage GPU BIOS, ISOs, Disks and other hardware parts dependencies to finally know how to create and configure the first virtual machine.

## Commands to Manage VMs

These are some basic commands to manage VMs:

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

If you want enable or disable autostart on VM when you power up the host machine, run the following command:

```bash
# Enable
virsh autostart ${NAME}

# Disable
virsh autostart --disable ${NAME}
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

## VGA BIOS Management

To avoid issues with GPU cards inside VMs, we recommend extracting the GPU ROM BIOS. If you know that your GPU card doesn't need it, just skip this process, otherwise, you need to run this just once.

Use GPU-z on Windows standalone machine to extract the ROM file for the GPU or download the BIOS at <https://www.techpowerup.com/vgabios/>. To extract the BIOS file from Linux, you must run a live USB distro with CLI mode, then run the following command:

```bash
echo 1 > /sys/bus/pci/devices/{$ID}/rom
cat /sys/bus/pci/devices/{$ID}/rom > ${NAME}.rom
echo 0 > /sys/bus/pci/devices/{$ID}/rom
```

After extracting, login on the main host and run the command below to copy the ROM to the correct location:

```bash
# IMPORTANT: Every GPU BIOS file must be located at /var/lib/libvirt/vbios/
# Create folder if not exists
mkdir -p /var/lib/libvirt/vbios/

# Copy ROM file from where you saved it
cp NAVI22.rom /var/lib/libvirt/vbios/

# Fix permissions on ROM files
chmod -R 660 /var/lib/libvirt/vbios/*
chown -R qemu:qemu /var/lib/libvirt/vbios
restorecon -R -vF /var/lib/libvirt/vbios
```

**NVIDIA only**: If you need to patch the GPU BIOS in order to avoid ``code 43`` issue, use the following command to create a patched ROM version of your GPU BIOS:

```bash
echo "`grep -aoib video ${NAME}.rom | \
  cut -d: -f1` / 16 * 16" | bc | \
  xargs -I OFF dd if=${NAME}.rom of=${NAME}-P.rom bs=OFF skip=1
```

## ISO Management

To install operational systems inside VMs, you probably need an ISO file. You can copy the ISOs from another hard drive or download the ISO file to the correct location as show below:

```bash
# IMPORTANT: Every ISO must be located at /var/lib/libvirt/images/
# Create folder if not exists
mkdir -p /var/lib/libvirt/images/

# Copy ISO file from where you saved it
cp ubuntu.iso /var/lib/libvirt/images/ubuntu.iso

# Download the ISO that you want
cd /var/lib/libvirt/images/
wget http://releases.ubuntu.com/jammy/ubuntu-22.04-desktop-amd64.iso

# Download virtio drivers for Windows
cd /var/lib/libvirt/images/
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso

# Fix permissions on ISO files
chmod -R 660 /var/lib/libvirt/images/*
chown -R qemu:qemu /var/lib/libvirt/images
restorecon -R -vF /var/lib/libvirt/images
```

## Disk Management

Virtual machines likely require a disk to install and run their software. Here, we are a few options like directly attaching a physical disk, creating a LVM partition or creating a virtual disk for this propose with ``.qcow2`` or ``.raw`` disk images format. Use one of the commands below to create the desired disk format for the VM:

```bash
# LVM partition
lvcreate -n ${NAME} -L 100G ${VOLUME_GROUP}

# Raw disk
qemu-img create -f raw /var/lib/libvirt/images/${NAME}.raw 100G

# QCOW2 disk
qemu-img create -f qcow2 /var/lib/libvirt/images/${NAME}.qcow2 100G

# Fix permissions on virtual disk files
chmod -R 660 /var/lib/libvirt/images/*
chown -R qemu:qemu /var/lib/libvirt/images
restorecon -R -vF /var/lib/libvirt/images
```

You don't need to format the disk or partition, just pass it to the VM and run the installation process.

## Creating a Virtual Machine from CLI

Now that you have solved the GPU BIOS, ISO and Disk dependencies for the VM, you can create and configure a new virtual machine. To finish, we just need to know the list of OS variants available to install and the hardware parts that can be attached to the VM:

```bash
osinfo-query os # OS list
virsh nodedev-list # PCI-E parts
lsusb # USB devices
lsblk # Disks
```

Isolate the relevant info and save such information to use later on the VM configuration.

To create a VM directly from CLI, use the commands as below. This process is slightly different from the method available everywhere because we will first generate one sample XML based on the described OS, tweak the settings and finally create a new VM based on the specs that we declared:

```bash
# Edit variables
CDROM="/var/lib/libvirt/images/ubuntu-22.04.iso"
DISK="/dev/hypervisor/ubuntu"
VARIANT="ubuntu22.04"
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

Now that you generated the basic XML, you need to edit the XML file to set extra configurations and tweak your VM configuration. The best configurations are described on the next topic: **[XML Configurations](5%20-%20XML%20Configurations.md)**:

```bash
vim ${NAME}.xml
```

After making the necessary changes on the XML configuration, import the XML file to create the virtual machine and you can start the VM:

```bash
virsh define ${NAME}.xml
virsh start ${NAME}
virsh console ${NAME}
```

That is all. Install the operating system and be happy!