# Configuring the Bridge Network

Setting up a bridge network will allow full communication of VMs with other devices in your network and also will allow communication between the host and VMs, and between VMs inside the host. As a nice example, if you want to allow your phone to communicate with a Windows VM for remote gaming, you will need a bridge network in your setup.

By default, Fedora does not activate any bridge network for KVM/QEMU, but you can run the following commands to create the bridge network with the host adapter. Before running the commands, please do not forget to replace the variables values to the ones that match your hardware:

**WARNING:** Once you run the following commands, the current connection will be deleted, so any SSH connections will be lost. We recommend running these commands on the host terminal.

```bash
# See current network details
sudo nmcli connection
sudo nmcli device
```

```bash
# Update with correct values 
CONNECTION_NAME="enp7s0"
NETWORK_INTERFACE="enp7s0"

# Create the bridge network
sudo nmcli connection delete virbr0
sudo nmcli connection add type bridge \
    autoconnect yes \
    con-name virbr0 \
    ifname virbr0 \
    ipv4.method auto \
    ipv6.method auto

# Put network interface inside the bridge
sudo nmcli connection delete "${CONNECTION_NAME}"
sudo nmcli connection add type bridge-slave \
    autoconnect yes \
    con-name "${NETWORK_INTERFACE}" \
    ifname "${NETWORK_INTERFACE}" \
    master virbr0

# Restore connection
sudo nmcli connection up virbr0
```

Now is time to update the KVM/QEMU network, so he can understand that ``virbr0`` is in fact a bridge:

```bash
# Update libvirt network
virsh net-destroy default
virsh net-undefine default
virsh net-define /dev/stdin <<EOF
<network>
  <name>default</name>
  <forward mode='bridge'/>
  <bridge name='virbr0' delay='0'/>
</network>
EOF

# Start network and check status
virsh net-autostart default
virsh net-start default
virsh net-list
```

If your motherboard also has WiFi and you want to completely disable it, just run the command below to do it. By disabling WiFi (a service not used at this case), you can avoid some bugs along the way reducing the need of system maintenance:

```bash
# Disable radio
sudo nmcli radio wifi off

# Stop service
sudo systemctl disable wpa_supplicant
sudo systemctl stop wpa_supplicant
```

Once everything has been completed, restart the system and your network is ready now:

```bash
sudo reboot
```

----

Next step: **[Enabling PCI-e Passthrough](02%20-%20PCI-e%20Passthrough.md)**
