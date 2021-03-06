# KVM / QEMU XML Configuration Tips

This is an extensive list of relevant configurations that you can do in your VM. It's a must read if you want setup hooks and configure your VM to work well. Please remember that they are just samples, you must replace the hardware parts to match yours and to match the specs of your VM.

Just as a reminder, you can edit the VM configurations for an already created VM using the following command:

```bash
virsh edit ${NAME}
```

## OS Optimizations

To set as full hardware virtualization mode, allow snapshots on UEFI (only to ``.qcow2`` disk format) and also enable the boot from CDROM, your specs should be something like:

```xml
<os>
  <type arch="x86_64" machine="q35">hvm</type>
  <loader readonly="yes" type="rom">/usr/share/edk2/ovmf/OVMF_CODE.fd</loader>
  <bootmenu enable="yes"/>
  <boot dev="hd"/>
  <boot dev="cdrom"/>
</os>
```

## CPU Optimizations

For a good CPU optimization, we first need to know how the CPU set cores and threads. Use the following command to understand how it works in your case. We need that information to configure the CPU pinning correctly:

```bash
# Look for <topology> section
virsh capabilities
```

As a rule of thumb, you must leave the first 1 or 2 CPU core(s) to the hypervisor host. The remaining cores should be distributed to the active VMs.

For a desktop VM you could use an 8-core setup like below (notice that ``cpuset`` match the topology of the CPU where 4 and 12 are sibling threads):

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
<cpu mode="host-passthrough" check="none">
  <topology sockets="1" cores="4" threads="2"/>
  <feature policy="require" name="topoext"/>
  <cache mode="passthrough"/>
</cpu>
```

In the case of a server VM, you could use a 4-core setup with the following syntax to isolate its cores from the desktop VM:

```xml
<vcpu cpuset="2,3,10,11">4</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="2"/>
  <vcpupin vcpu="1" cpuset="3"/>
  <vcpupin vcpu="2" cpuset="10"/>
  <vcpupin vcpu="3" cpuset="11"/>
</cputune>
<cpu mode="host-passthrough" check="none">
  <topology sockets="1" cores="2" threads="2"/>
  <feature policy="require" name="topoext"/>
  <cache mode="passthrough"/>
</cpu>
```

You can also increase the performance of CPU by setting the performance mode on host. To enable it, just declare scale CPU hook:

```xml
<description>--scale-cpu</description>
```

You also can configure CPU pinning to fully isolate CPUs and increase performance on VM. On the case below, we are allowing the host to use only the first 4 threads when VM starts and then restore all 16 threads to the host when the VM stops:

```xml
<description>--cpu-host-allow=0,1,8,9 --cpu-host-restore=0-15</description>
```

## Linux Enhancements

These features are also configured to bypass Nvidia graphics card issues. It's recommended for Linux VMs only, if you are virtualizing Windows, see the next section:

```xml
<features>
  <acpi/>
  <apic/>
  <kvm>
    <hidden state="on"/>
  </kvm>
  <hyperv>
    <relaxed state="on"/>
    <spinlocks retries="8191" state="on"/>
    <vapic state="on"/>
    <vendor_id state="on" value="1234567890ab"/>
  </hyperv>
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
  <kvm>
    <hidden state="on"/>
  </kvm>
  <hyperv>
    <relaxed state="on"/>
    <vapic state="on"/>
    <spinlocks state="on" retries="8191"/>
    <vpindex state="on"/>
    <synic state="on"/>
    <stimer state="on"/>
    <reset state="on"/>
  </hyperv>
  <vmport state="off"/>
  <ioapic driver="kvm"/>
</features>
<clock offset="localtime">
  <timer name="rtc" tickpolicy="catchup"/>
  <timer name="pit" tickpolicy="delay"/>
  <timer name="hpet" present="no"/>
  <timer name="hypervclock" present="yes"/>
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
  <suspend-to-mem enabled='no'/>
  <suspend-to-disk enabled='no'/>
</pm>
```

## CDROM device for ISO

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

After the OS has been installed, remember to remove the ``<source>`` on the CDROM device or remove the full disk entry.

## Physical Disks

You can add a physical disks into VMs. This is very useful if you have and disk with Ext-FAT or NTFS format. First, make sure that the target disk is unused elsewhere since a physical disk can be used only in one VM:

```xml
<devices>
  <disk type="block" device="disk">
    <driver name="qemu" type="raw" cache="writeback" io="io_uring" discard="unmap"/>
    <source dev="/dev/sda"/>
    <target dev="sdb" bus="virtio"/>
  </disk>
</devices>
```

**TIP:** the same entry format can be used to attach LVM partitions.

## GPU Passthrough and VGA Bios

**IMPORTANT**: To see boot menu, you must use the console entry on Cockpit or with the command ``virsh console ${NAME}``.

Once you have the ROM file configured on the hypervisor system (if necessary), edit the VM config and place the following settings to passthrough the GPU:

```xml
<devices>
  <!-- Nvidia GPU -->
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x09" slot="0x00" function="0x0"/>
    </source>
    <rom file="/var/lib/libvirt/vbios/GP104-P.rom"/>
    <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x0"/>
  </hostdev>
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x09" slot="0x00" function="0x1"/>
    </source>
    <rom file="/var/lib/libvirt/vbios/GP104-P.rom"/>
    <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x1"/>
  </hostdev>

  <!-- AMD GPU -->
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x08" slot="0x00" function="0x0"/>
    </source>
    <rom file="/var/lib/libvirt/vbios/NAVI14.rom"/>
    <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x0"/>
  </hostdev>
  <hostdev mode="subsystem" type="pci" managed="yes">
    <source>
      <address domain="0x0000" bus="0x08" slot="0x00" function="0x1"/>
    </source>
    <rom file="/var/lib/libvirt/vbios/NAVI14.rom"/>
    <address type="pci" domain="0x0000" bus="0x07" slot="0x00" function="0x1"/>
  </hostdev>
</devices>
```

If you have only one GPU, don`t forget to declare the main GPU passthrough hook option:

```xml
<description>--main-gpu-passthrough</description>
```

And if you are running an AMD GPU, we recommend set the GPU reset on such device:

```xml
<description>--gpu-reset=0000:08:00.0</description>
```

This was everything that I need to make my GPUs works. In my experience, I don't need to remove the VNC display after OS installation, it will work as a boot display only to manage the boot options.

If you are having problems with the black screen in your VM, use the VNC display to check what is the issue. You can attach a VNC display if necessary with the following XML:

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

In the case of problems, you should remove the VNC display once the GPU driver has been installed on VM because it may prevent the GPU passthrough to run properly. Don't forget also to remove any line related to ``<graphics>``, ``<video>`` and other unnecessary devices from your VM config in the XML (SPLICE or VNC related). This step can solve a lot of your problems!

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

## Extended Mouse and Keyboard support

Make sure your VM contains the following lines to extend the support for mouse and keyboard:

```xml
<devices>
  <input type='mouse' bus='virtio'>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x0e' function='0x0'/>
  </input>
  <input type='keyboard' bus='virtio'>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x0f' function='0x0'/>
  </input>
  <input type='mouse' bus='ps2'/>
  <input type='keyboard' bus='ps2'/>
</devices>
```

Also, when using mouse and keyboard, make sure to remove the ``tablet`` input interface. It will reduce CPU usage and bring more performance:

```xml
<!-- REMOVE -->
<devices>
  <input type="tablet" bus="usb">
    <address type="usb" bus="0" port="1"/>
  </input>
</devices>
```

## USB Devices

You should passthrough USB devices like mouse and keyboard to direct VM access if they are ALWAYS connected to the host and will be connected to this specific VM when it starts. Use ``lsusb`` to know the vendor and product IDs then fill the XML like below:

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

For other devices like USB drivers, gamepads, headphones, and many more that are often connected to the host or can be switched between VMs, I recommend setting it to be plugged in live with the hook option. That way, any USB device attached to any USB port will be automatically redirected to the VM:

```xml
<description>--usb-passthrough=all</description>
```

Please keep in mind that you must only plug in the device after VM has been started to trigger the hook. If the device should start connected on the VM, then you must setup it XML like described before (USB mouse and keyboard on Windows and MacOS VMs need this).

## Bridge Network

Use the following model to setup an extra bridge network if necessary:

```xml
<devices>
  <interface type="bridge">
    <mac address="52:54:00:72:58:ce"/>
    <source bridge="br0"/>
    <model type="virtio"/>
    <address type="pci" domain="0x0000" bus="0x01" slot="0x00" function="0x0"/>
  </interface>
</devices>
```

If you are using Windows, is recommended change the model type from ``virtio`` to ``e1000e``, but ``virtio`` can have more performance:

```xml
<model type="e1000e"/>
```

On MacOs, the recommended model type is ``vmxnet3``:

```xml
<model type="vmxnet3"/>
```

## QEMU Guest Agent

The QEMU guest agent add many cool features to the VM, including more precise VM statistics and better VM control. Use the following configuration inside the VM settings to enable the guest agent:

```xml
<devices>
  <channel type="unix">
    <target type="virtio" name="org.qemu.guest_agent.0"/>
    <address type="virtio-serial" controller="0" bus="0" port="1"/>
  </channel>
</devices>
```

After VM is running, you must install the QEMU guest agent to increase performance and get more precise metrics. On Windows, download and use the [VirtIO ISO](https://github.com/virtio-win/virtio-win-pkg-scripts). On Linux, install the ``qemu-guest-agent`` package. MacOS does not have a compatible package yet.

## Serial Console Access

You can make the command ``virsh console`` result in directly access to the serial console inside virtual machines. First, make sure the serial console is configured:

```xml
<serial type='pty'>
  <target type='isa-serial' port='0'>
    <model name='isa-serial'/>
  </target>
</serial>
<console type='pty'>
  <target type='serial' port='0'/>
</console>
```

To enable such feature, you need to enable the serial service **INSIDE** the VM (not in the host) once the VM has been installed. For example:

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

If you come from the previous topic, set the relevant XML details for you VM and continue the VM creation.

I will update this configuration guide once I learn more relevant details in the future. If you are targeting one specific type of virtual machine, I created a few topics on that to write my observations:

- **[Linux Virtualization](5.1%20-%20Linux.md)**
- **[Windows Virtualization](5.2%20-%20Windows.md)**
- **[MacOS Virtualization](5.3%20-%20MacOS.md)**
- **[Android Virtualization](5.4%20-%20Android.md)**
- **[Chrome OS Virtualization](5.5%20-%20Chrome%20OS.md)**
