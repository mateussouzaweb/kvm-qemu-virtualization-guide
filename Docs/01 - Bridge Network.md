# Configuring the Bridge Network

By default, Fedora does not create any bridge network for KVM/QEMU on installation. We recommend setting up a bridge network because this will allow full communication of VMs with other devices in your network and also will allow communication between the host and VMs, and between VMs inside the host.

As a nice example, if you want to allow your phone to communicate with a Windows VM for remote gaming, you will need a bridge network in your setup.

Run the following commands to create the bridge network with the host adapter, do not forget to replace the variables values to the ones that match your hardware:

**WARNING:** Once you run the following commands, the current connection will be deleted, so any SSH connections will be lost. We recommend running these commands on the host terminal.

```bash
# See current network details
sudo nmcli connection
sudo nmcli device
```

```bash
# Update with correct values 
CONNECTION_NAME="enp5s0"
NETWORK_INTERFACE="enp5s0"

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

# Restart
sudo reboot
```

If your motherboard also has WiFi and you want to completely disable it, just run the command below to do it. By disabling WiFi (a service not used at this case), you can avoid some bugs along the way reducing the need of system maintenance:

```bash
# Disable radio
sudo nmcli radio wifi off

# Stop service
sudo systemctl disable wpa_supplicant
sudo systemctl stop wpa_supplicant
```

Very good! Your network is ready now.

----

Next step: **[Enabling PCI-e Passthrough](02%20-%20PCI-e%20Passthrough.md)**
