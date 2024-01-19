# File Sharing

When you have many virtual machines, file sharing can start becoming a problem to manage with the mount of disks inside VMs. You can surely create an advanced VM for NAS using great platforms like TrueNAS but if you think that is too much for you setup, you can have a simple file sharing service in the host system that can be access simultaneously in all places, including VMs and others devices in in your network like phones and laptops.

Follow the steps below to learn how to create it using Samba File Sharing service.

## Sharing Files with Samba Sharing

Samba is a file sharing protocol compatible with Linux, Windows and MacOS that requires no additional software to work. To enable this feature, we will install the service on the host and configure it. Start by installing the software and enabling it on firewall:

```bash
dnf install -y samba
systemctl enable smb --now
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
```

Now, we will create the user authentication details that will able to connect on this shared location:

```bash
# Add samba user
USERNAME="mateussouzaweb"
smbpasswd -a ${USERNAME}
```

Once we have the user, create a LVM partition to place shared files, format it and mount on the system:

```bash
# Create partition
VOLUME_GROUP="hypervisor"
lvcreate -n files -L 100G ${VOLUME_GROUP}

# Format the partition
mkfs.xfs /dev/${VOLUME_GROUP}/files

# Persist and mount volume
echo "/dev/${VOLUME_GROUP}/files /mnt/files xfs defaults 0 0"  >> /etc/fstab
mkdir /mnt/files
chown -R ${USERNAME}:${USERNAME} /mnt/files
mount -a

# Set SELinux permissions
semanage fcontext --add --type "samba_share_t" "/mnt/files(/.*)?"
restorecon -R -vF /mnt/files
```

Finally, you just need to configure the samba service and restart it:

```bash
# Write config
cat <<EOT > /etc/samba/smb.conf
[global]
    workgroup = SAMBA
    security = user
    passdb backend = tdbsam

[files]
    path = /mnt/files
    comment = Files
    valid users = ${USERNAME}
    writeable = Yes
    browseable = Yes
    read only = No
    inherit acls = Yes
    create mask = 0644
    directory mask = 0775
    write list = user
EOT

# Restart service to changes to take effect
systemctl restart smb
```

Great! You can now configure and open the shared location on VMs and other machines.

### Connecting to a Samba Shared Location

**Windows:** Open the "File Explorer" and then right-click on "This PC" (in the left pane). From the resulting context menu, select "Add A Network Location". When requested for the network address, use ``\\192.168.0.100\files`` and continue by filling in the authentication details.

**MacOS:** Open "Finder" and use the menu option "Go" > "Connect to Server...". In the address field, enter ``smb://192.168.0.100/files`` and click on "Connect". Accept the prompt and continue by filling in the authentication details when requested.

**Linux:** In the "Connect to Server" option of the file browser, enter the address as ``smb://192.168.0.100/files`` and continue by filling in the authentication details.

**Docker:** You also can mount samba share volumes with the CIFS protocol. Check the documentation for how to do it.
