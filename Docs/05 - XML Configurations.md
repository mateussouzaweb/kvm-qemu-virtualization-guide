# KVM / QEMU XML Configuration Tips

This is an extensive list of relevant configurations that you can do in your VM. It's a must read if you want to setup hooks and configure your VM to work well. Please remember that they are just samples, you must replace the hardware parts to match yours and to match the specs of your VM. You can also check the ``Samples/`` folder to see some final examples.

Just as a reminder, you can edit the XML file or the VM configurations for an already created VM using the following commands:

```bash
# XML configuration
vim ${NAME}.xml

# Already created VM
virsh edit ${NAME}
```

## OS Optimizations

To set as full hardware virtualization mode, allow snapshots on UEFI (only to ``.qcow2`` disk format) and also enable the boot from CDROM, your specs should be something like:

```xml
<os firmware="efi">
  <type arch="x86_64" machine="q35">hvm</type>
  <firmware>
    <feature enabled="yes" name="secure-boot"/>
    <feature enabled="yes" name="enrolled-keys"/>
  </firmware>
  <bootmenu enable="yes"/>
  <boot dev="hd"/>
  <boot dev="cdrom"/>
</os>
```

Please be aware that these settings enable NVRAM and if you edit the machine details, Secure Boot can return error when starting the machine. To fix that issue, disable Secure Boot or start the VM with the reset NVRAM option:

```bash
virsh start ${NAME} --reset-nvram
```

## CPU Optimizations

For a good CPU optimization, we first need to know how the CPU set cores and threads. Use the following command to understand how it works in your case. We need that information to configure the CPU pinning correctly:

```bash
# Look for <topology> section
virsh capabilities
```

As a rule of thumb, you must leave the first 1 or 2 CPU core(s) to the hypervisor host. The remaining cores should be distributed to the active VMs.

For a desktop VM you could use an 4-core and 8-threads setup like below (notice that ``cpuset`` match the topology of the CPU where 4 and 12 are sibling threads):

```xml
<vcpu cpuset="4-7,12-15">8</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="4"/>
  <vcpupin vcpu="1" cpuset="5"/>
  <vcpupin vcpu="2" cpuset="6"/>
  <vcpupin vcpu="3" cpuset="7"/>
  <vcpupin vcpu="4" cpuset="12"/>
  <vcpupin vcpu="5" cpuset="13"/>
  <vcpupin vcpu="6" cpuset="14"/>
  <vcpupin vcpu="7" cpuset="15"/>
</cputune>
<cpu mode="host-passthrough" check="none" migratable="off">
  <topology sockets="1" cores="4" threads="2"/>
  <feature policy='require' name='invtsc'/>
  <feature policy="require" name="topoext"/>
  <cache mode="passthrough"/>
</cpu>
```

In the case of running a another VM in parallel, you could use a 2-core and 4-threads setup with the following syntax to isolate its cores from the desktop VM:

```xml
<vcpu cpuset="2,3,10,11">4</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="2"/>
  <vcpupin vcpu="1" cpuset="3"/>
  <vcpupin vcpu="2" cpuset="10"/>
  <vcpupin vcpu="3" cpuset="11"/>
</cputune>
<cpu mode="host-passthrough" check="none" migratable="off">
  <topology sockets="1" cores="2" threads="2"/>
  <feature policy='require' name='invtsc'/>
  <feature policy="require" name="topoext"/>
  <cache mode="passthrough"/>
</cpu>
```

Notice that the cores 0, 1, 8 and 9 (the first 4 threads) were not declared on such XMLs because they will be reserved to the host. We also should use the virtualization hooks to declare such cores as preserved to fully isolate CPUs from the host and scale CPU to performance mode on host to increase responsiveness on running VMs:

```xml
<description>--cpu-scaling-mode=performance --preserve-cores=0,1,8,9</description>
```

## Linux Enhancements

These features are also configured to bypass Nvidia graphics card issues. It's recommended for Linux VMs only, if you are virtualizing Windows, see the next section:

```xml
<features>
  <acpi/>
  <apic/>
  <hyperv>
    <relaxed state="on"/>
    <vapic state="on"/>
    <spinlocks state="on" retries="8191"/>
    <vendor_id state="on" value="1234567890ab"/>
  </hyperv>
  <kvm>
    <hidden state="on"/>
  </kvm>
  <vmport state="off"/>
  <ioapic driver="kvm"/>
</features>
```

## Windows Enhancements

If the VM is running Windows, you can add some configurations to save CPU usage and increase the VM responsiveness:

```xml
<features>
  <acpi/>
  <apic/>
  <pae/>
  <hyperv>
    <relaxed state="on"/>
    <reset state="on"/>
    <synic state="on"/>
    <spinlocks state="on" retries="8191"/>
    <stimer state="on"/>
    <vapic state="on"/>
    <vendor_id state="on" value="1234567890ab"/>
    <vpindex state="on"/>
  </hyperv>
  <kvm>
    <hidden state="on"/>
  </kvm>
  <ioapic driver="kvm"/>
  <vmport state="off"/>
</features>
<clock offset="localtime">
  <timer name="rtc" tickpolicy="catchup"/>
  <timer name="pit" tickpolicy="delay"/>
  <timer name="hpet" present="no"/>
  <timer name="hypervclock" present="yes"/>
  <timer name="tsc" present="yes" mode="native"/>
</clock>
```

## MacOS Enhancements

MacOS needs just a tiny list of features. This kind of VM has its own hacks to make it work and will be presented in their exclusive topic later:

```xml
<features>
  <acpi/>
  <apic/>
</features>
```

## Power Events

Just tells what to do when events are triggered and declare suspend rules. Add it to XML if missing:

```xml
<on_poweroff>destroy</on_poweroff>
<on_reboot>restart</on_reboot>
<on_crash>destroy</on_crash>
<pm>
  <suspend-to-mem enabled="no"/>
  <suspend-to-disk enabled="no"/>
</pm>
```

## CDROM Device from ISO

You can manually attach ISO as CDROM device to run installations. This also can be used in the future if you want to reinstall a distro or try something else:

```xml
<devices>
  <disk type="file" device="cdrom">
    <driver name="qemu" type="raw"/>
    <source file="/var/lib/libvirt/images/ubuntu.iso"/>
    <target dev="sdc" bus="sata"/>
    <readonly/>
  </disk>
</devices>
```

On Windows, you also will need to put an additional CDROM device for VirtIO packages:

```xml
<devices>
  <disk type="file" device="cdrom">
    <driver name="qemu" type="raw"/>
    <source file="/var/lib/libvirt/images/virtio-win.iso"/>
    <target dev="sdd" bus="sata"/>
    <readonly/>
  </disk>
</devices>
```

After the OS has been installed, remember to remove the ``<source>`` on the CDROM device or just remove the full ``<disk>`` entry.

## Physical Disks and LVM Partitions

You can add a physical disk or LVM partitions into VMs. First, make sure that the target device is not used elsewhere since a physical disk or LVM partition can be used only in one running VM:

```xml
<!-- Physical Disk -->
<devices>
  <disk type="block" device="disk">
    <driver name="qemu" type="raw" cache="writeback" io="io_uring" discard="unmap"/>
    <source dev="/dev/sda"/>
    <target dev="sdb" bus="virtio"/>
  </disk>
</devices>

<!-- LVM Partition -->
<devices>
  <disk type="block" device="disk">
    <driver name="qemu" type="raw" cache="writeback" io="io_uring" discard="unmap"/>
    <source dev="/dev/hypervisor/files"/>
    <target dev="sdb" bus="virtio"/>
  </disk>
</devices>
```

## Virtual Disks

You can add RAW or QCOW2 virtual disks into VMs. First, make sure that the target virtual disk is unused elsewhere since a disk can be used only in one running VM:

```xml
<!-- RAW -->
<devices>
  <disk type="file" device="disk">
    <driver name="qemu" type="raw" cache="writeback" io="io_uring" discard="unmap"/>
    <source file="/var/lib/libvirt/images/${NAME}.raw"/>
    <target dev="sdb" bus="virtio"/>
  </disk>
</devices>

<!-- QCOW2 -->
<devices>
  <disk type="file" device="disk">
    <driver name="qemu" type="qcow2" cache="writeback" io="io_uring" discard="unmap"/>
    <source file="/var/lib/libvirt/images/${NAME}.qcow2"/>
    <target dev="sdb" bus="virtio"/>
  </disk>
</devices>
```

## GPU Passthrough and VGA Bios

Once you have the ROM file configured on the hypervisor system, edit the VM config and place the following settings to passthrough the GPU to the VM:

```xml
<devices>
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x0c" slot="0x00" function="0x0"/>
    </source>
    <rom file="/var/lib/libvirt/vbios/NAVI22.rom"/>
    <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x0"/>
  </hostdev>
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x0c" slot="0x00" function="0x1"/>
    </source>
    <rom file="/var/lib/libvirt/vbios/NAVI22.rom"/>
    <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x1"/>
  </hostdev>
</devices>
```

Since we are attaching the GPU to the VM, we must set the virtualization hook options with such reference:

```xml
<!-- If you have only one GPU or is attaching the main GPU -->
<description>--gpu-passthrough=main,0c:00.0,0c:00.1</description>

<!-- If you are attaching other secondary GPU -->
<description>--gpu-passthrough=secondary,08:00.0,08:00.1</description>
```

This is everything that I need to make my GPUs work. In my experience, you usually don't need to remove the VNC display after OS installation, it will work as a boot display only to manage the boot options.

If you are having problems with the black screen in your VM, use the VNC display to check what the issue is. You can attach a VNC display, if necessary, with the following XML:

```xml
<devices>
  <graphics type="vnc" port="-1" autoport="yes" listen="0.0.0.0">
    <listen type="address" address="0.0.0.0"/>
  </graphics>
  <video>
    <model type="bochs" vram="16384" heads="1" primary="yes"/>
    <address type="pci" domain="0x0000" bus="0x00" slot="0x01" function="0x0"/>
  </video>
</devices>
```

In the case of problems, you should remove the VNC display once the GPU driver has been installed on VM because it may prevent the GPU passthrough from running properly. Don't forget also to remove any line related to ``<graphics>``, ``<video>`` and other unnecessary devices from your VM config in the XML (SPLICE or VNC related). This step can solve a lot of your problems!

## Audio Passthrough

This will attach the system onboard audio PCI device directly to the VM, so microphone and other peripherals will work as expected inside the VM:

```xml
<devices>
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x0b" slot="0x00" function="0x3"/>
    </source>
    <address type="pci" domain="0x0000" bus="0x09" slot="0x00" function="0x0"/>
  </hostdev>
</devices>
```

I do not recommend attaching audio with audio drivers from Linux, but you can try if you want.

## Extended Mouse and Keyboard Support

Make sure your VM contains the following lines to extend the support for mouse and keyboard (they may be automatically expanded by the KVM):

```xml
<devices>
  <input type="mouse" bus="virtio"/>
  <input type="keyboard" bus="virtio"/>
  <input type="mouse" bus="usb"/>
  <input type="keyboard" bus="usb"/>
  <input type="mouse" bus="ps2"/>
  <input type="keyboard" bus="ps2"/>
</devices>
```

Also, when using mouse and keyboard, you can remove the ``tablet`` input interface if you will not use the virtual display interface with VNC - when using emulated display, the tablet input is better because mouse movement is more precise with the available area. When this is not necessary, you can remove it to will reduce CPU usage and bring more performance:

```xml
<devices>
  <!-- Add or remove interface -->
  <input type="tablet" bus="usb"/>
</devices>
```

## USB Devices

The best way to use USB devices inside VMs is by passing the entire USB Controller via its PCI-e interface. If you can use this method, that is everything that you need to do in order to passthrough any USB device to the VM:

```xml
<devices>
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x08" slot="0x00" function="0x1"/>
    </source>
  </hostdev>
<devices>
```

For others cases, you should configure USB devices like mouse and keyboard to direct VM access if they are ALWAYS connected to the host and will be connected to this specific VM when it starts. If you want to pass only specific USB devices, use ``lsusb`` to know the vendor and product IDs of these devices then fill the XML like below:

```xml
<devices>
  <hostdev mode="subsystem" type="usb" managed="yes">
    <source startupPolicy="optional">
      <vendor id="0x046d"/>
      <product id="0xc52b"/>
    </source>
  </hostdev>
</devices>
```

For other devices like USB drivers, gamepads, headphones, and many more that are often connected to the host or can be switched between VMs, I recommend setting it to be plugged in live with the virtualization hook option. That way, any USB device attached to any USB port will be automatically redirected to the VM:

```xml
<description>--usb-passthrough=all</description>
```

Please keep in mind that the virtualization hook method may not work in some specific machines and you must only plug in the device after VM has been started to trigger the hook. If the device should start connected on the VM, then you must setup its XML like described before (USB mouse and keyboard on Windows and MacOS VMs need this).

Also, it is important to notice that some USB devices can present detection and power issues because of the way they operate inside the host - this is special true for USB Bluetooth devices in motherboards. In such cases, you may need to change the QEMU USB device mapping format by adding the following XML tag:

```xml
<!-- Make sure domain contains the XML namespace for QEMU -->
<domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0">

  <!-- Append this entry -->
  <qemu:commandline>
    <qemu:arg value="-device"/>
    <qemu:arg value="usb-host,hostbus=1,hostaddr=3,bus=usb.0,port=3"/>
  </qemu:commandline>
</domain>
```

## TPM Device

A Trusted Platform Module (TPM) offers advanced protection with integrated cryptographic keys directly from the hardware. If your hardware contains the TPM device, you can passthrough it, but if not, you can still use an emulated one to get the same features on virtual machines. Both cases are show in the following XML:

```xml
<devices>
  <!-- Passthrough -->
  <tpm model='tpm-tis'>
    <backend type='passthrough'>
      <device path='/dev/tpm0'/>
    </backend>
  </tpm>

  <!-- Emulated -->
  <tpm model='tpm-tis'>
    <backend type='emulator' version='2.0'/>
  </tpm>
</devices>
```

## Bridge Network

Use the following model to setup the bridge network if necessary:

```xml
<devices>
  <interface type="bridge">
    <mac address="52:54:00:72:58:ce"/>
    <source bridge="virbr0"/>
    <model type="virtio"/>
    <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
  </interface>
</devices>
```

On MacOs, the recommended model type is ``vmxnet3``:

```xml
<model type="vmxnet3"/>
```

## QEMU Guest Agent

The QEMU guest agent adds many cool features to the VM, including more precise VM statistics and better VM control. Use the following configuration inside the VM settings to enable the guest agent:

```xml
<devices>
  <channel type="unix">
    <target type="virtio" name="org.qemu.guest_agent.0"/>
    <address type="virtio-serial" controller="0" bus="0" port="1"/>
  </channel>
</devices>
```

After VM is running, you must install the QEMU guest agent to increase performance and get more precise metrics. On Windows, download and use the [VirtIO ISO](https://github.com/virtio-win/virtio-win-pkg-scripts). On Linux, install the ``qemu-guest-agent`` package. MacOS does not have a compatible package yet.

```bash
sudo apt install -y qemu-guest-agent
systemctl start qemu-guest-agent
```

## SPICE Guest Agent

The SPICE project adds support for remote access to virtual machines in a seamless way. With SPICE, you can copy and paste between the host and the virtual machine, play videos, record audio, share usb devices and share folders without complications. Use the following settings inside the VM to enable the communication channel:

```xml
<channel type='spicevmc'>
   <target type='virtio' name='com.redhat.spice.0'/>
</channel>
```

After VM is running, you must install the SPICE guest agent. The VirtIO ISO already includes it on Windows. On Linux, install the ``spice-vdagent`` package. MacOS does not have a compatible package yet.

```bash
sudo apt install -y install spice-vdagent
systemctl start spice-vdagent
```

## Serial Console Access

You can make the command ``virsh console`` result in directly access to the serial console inside virtual machines. First, make sure the serial console is configured:

```xml
<serial type="pty">
  <target type="isa-serial" port="0">
    <model name="isa-serial"/>
  </target>
</serial>
<console type="pty">
  <target type="serial" port="0"/>
</console>
```

To enable such a feature on the VM, you need to enable the serial service **INSIDE** the VM (not in the host) once the VM has been installed. For example:

```bash
sudo systemctl enable --now serial-getty@ttyS0.service
```

Once the service is running inside the VM use the console command to access the terminal from host terminal:

```bash
# Press ENTER once its show the connected message
virsh console ${NAME}
```

Done!

----

If you come from the previous topic, set the relevant XML details for your VM and continue the VM creation.

I will update this configuration guide once I learn more relevant details in the future. If you are targeting one specific type of virtual machine, I created a few topics on that to write my observations:

- **[Linux Virtualization](06%20-%20Linux%20Virtualization.md)**
- **[Windows Virtualization](07%20-%20Windows%20Virtualization.md)**
- **[MacOS Virtualization](08%20-%20MacOS%20Virtualization.md)**
