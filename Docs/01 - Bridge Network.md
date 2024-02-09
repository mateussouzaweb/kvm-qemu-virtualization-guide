# Configuring the Bridge Network

By default, Fedora does not create any bridge network for KVM/QEMU on installation. We recommend setting up a bridge network because this will allow full communication of VMs with other devices in your network and also will allow communication between the host and VMs, and between VMs inside the host.

As a good example, if you want to allow your phone to communicate with a Windows VM for remote gaming, you will need a bridge network in your setup.

Run the following commands to create the bridge network with the host adapter, do not forget to replace the variables values to the ones that match your hardware:

**WARNING:** Once you run the following commands, the current connection will be deleted, so any SSH connections will be lost. We recommend running these commands on the host terminal.

```bash
# See current network details
nmcli

# Define variables
# Use the current network details to fill these variables
BRIDGE_NAME="virbr0"
NETWORK_INTERFACE="enp5s0"
SUBNET_IP="192.168.1.100/24"
GATEWAY="192.168.1.1"
DNS="192.168.1.1"

# Delete libvirt bridge if available
nmcli connection delete virbr0

# Create bridge
nmcli connection add type bridge \
    autoconnect yes con-name "${BRIDGE_NAME}" ifname "${BRIDGE_NAME}"

nmcli connection modify "${BRIDGE_NAME}" ipv4.addresses "${SUBNET_IP}" ipv4.method manual
nmcli connection modify "${BRIDGE_NAME}" ipv4.gateway "${GATEWAY}"

nmcli connection modify "${BRIDGE_NAME}" ipv4.ignore-auto-dns yes
nmcli connection modify "${BRIDGE_NAME}" ipv4.dns "${DNS}"

# Put network interface inside the bridge
nmcli connection delete "${NETWORK_INTERFACE}"
nmcli connection add type bridge-slave \
    autoconnect yes con-name "${NETWORK_INTERFACE}" ifname "${NETWORK_INTERFACE}" master "${BRIDGE_NAME}"

# Restore connection
nmcli connection up "${BRIDGE_NAME}"
nmcli connection show "${BRIDGE_NAME}"

# Restart network manager
systemctl restart NetworkManager
```

If your motherboard also has WiFi and you want to completely disable it, just run the command below to do it. By disabling WiFi (a service not used in our case), you can avoid some bugs along the way reducing the need of system maintenance:

```bash
# Disable radio
nmcli radio wifi off

# Stop service
systemctl disable wpa_supplicant
systemctl stop wpa_supplicant
```

Very good! Your network is ready now.

----

Next step: **[Enabling PCI-e Passthrough](02%20-%20PCI-e%20Passthrough.md)**
