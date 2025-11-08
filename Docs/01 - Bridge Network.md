# Configuring the Bridge Network

Setting up a bridge network will allow full communication of VMs with other devices in your network and also will allow communication between the host and VMs, and between VMs inside the host. As a nice example, if you want to allow your phone to communicate with a Windows VM for remote gaming, you will need a bridge network in your setup.

By default, Fedora and Ubuntu does not activate any bridge network for KVM/QEMU, but you can follow this guide to know details about our connection and create the bridge network with the host adapter:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# See current network details
sudo nmcli connection
sudo nmcli device
```

Now that you know the details about your network, let's modify the connection mode of the host machine. Before running the commands, please do not forget to replace the variables values to the ones that match your hardware:

**WARNING:** Once you run the following commands, the current connection will be deleted, so any SSH connections will be lost. We recommend running these commands on the host terminal.

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER / ATOMIC:**

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

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu uses a different configuration though netplan:
# You need to edit the file to have something like below
sudo vim /etc/netplan/50-bridge-network.yaml
```

```yml
# Make sure to put the correct connection name here
network:
  version: 2
  bridges:
    virbr0:
      dhcp4: true
      dhcp6: true
      interfaces:
        - ${CONNECTION_NAME}
```

Reboot the system and the bridge is completed.

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# WARNING:
# - Once you reboot, IP address will be changed!
# - Make sure router is not forcing static IP on these machines..
# - You may need to configure DNS IPs again...
sudo reboot
sudo ip link show
```

## Bridge Network on KVM/QEMU

Now is time to update the KVM/QEMU network, so he can understand that ``virbr0`` is in fact a bridge and use that as default connection mode:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

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

If your motherboard also has WiFi and you want to completely disable it, just run the command below to disable it. By disabling WiFi (a service not used at this case), you can avoid some bugs along the way reducing the need of system maintenance:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Disable radio
sudo nmcli radio wifi off

# Stop service
sudo systemctl disable wpa_supplicant
sudo systemctl stop wpa_supplicant
```

Once everything has been completed, restart the system and your network is ready now:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
sudo reboot
```

Once the network is working nicely, we can start tweaking the PCI-e passthrough for better yet virtual machines.

----

Next step: **[Enabling PCI-e Passthrough](02%20-%20PCI-e%20Passthrough.md)**
