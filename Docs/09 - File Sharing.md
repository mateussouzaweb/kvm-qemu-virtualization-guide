# File Sharing

When you have many virtual machines or need to share files with others devices, real time file sharing starts to become a necessity... sharing files is an extensive topic and has many ways to solve it.

The mainstream option is to create a VM for NAS software using platforms like TrueNAS with exclusively attached disks to that VM. This can be overkill for some setups, so, we present a few reliable alternatives that are less resource consuming and simple to setup.

## Sharing Files with VirtIOFS

**Important:** This method works only for local virtual machines and cannot be accessed outside the hypervisor. If you want to share files outside the hypervisor, use the next sharing option (Samba Sharing).

[VirtIOFS](https://virtio-fs.gitlab.io/) is a shared file system that lets virtual machines access a specific folder on the hypervisor. Unlike existing approaches that generally offer network wide file sharing, VirtIOFS is designed to offer local only file system to virtual machines and as result, performance is phenomenal.

The activation process is very easy. Start by editing the VM and add the following extra syntax with the mount specifications:

```bash
virsh edit ${NAME}
```

```xml
<domain>
  <!-- Configure memory backing -->
  <memoryBacking>
    <source type='memfd'/>
    <access mode='shared'/>
  </memoryBacking>

  <!-- Add mount paths -->
  <devices>
    <filesystem type='mount' accessmode='passthrough'>
      <driver type='virtiofs' queue='1024'/>
      <source dir='/path/to/files'/>
      <target dir='files'/>
    </filesystem>
  </devices>
</domain>
```

Save changes and start the virtual machine. 

On Windows VMs, make sure to have the VirtIO packages installed. Additional drivers should be added automatically to the "*File Explorer*" in the "*Drivers and Devices*" section.

On Linux VMs, once the virtual machine is started, create the directory and mount the new shared FS. Here are the basic instructions:

```bash
# First, make sure path exists
mkdir /mnt/files

# Temporary mount
sudo mount -t virtiofs files /mnt/files

# Persistent mount on each restart
echo "files /mnt/files virtiofs defaults 0 0" | sudo tee -a /etc/fstab
```

On MacOS, you can mount the shared FS with the following command:

```bash
# First, make sure path exists
mkdir $HOME/files

# Temporary mount
# For persistent mount, use Automator with AppleScript 
mount_virtiofs files $HOME/files
```

Remember, this method works only on local virtual machines. If you need to share files with other devices, please consider the Samba Sharing service option.

## Sharing Files with Samba Sharing

Samba sharing uses the network layer to work and has the benefit of offering a location that can be accessed simultaneously in all places, including VMs and other devices in your network like phones and laptops. Samba is compatible with Linux, Windows and MacOS that requires no additional software to work. 

To enable this feature on the hypervisor, start by installing the software and enabling it on firewall:

```bash
dnf install -y samba
systemctl enable smb --now
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
```

Now, create the user authentication details that will able to connect on the shared location (please note that the user authentication details mentioned here is not the same as the hypervisor user):

```bash
# Add samba user
USERNAME="mateussouzaweb"
smbpasswd -a ${USERNAME}
```

Once you have created the user, create a LVM partition to place shared files, format it and mount on the system:

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

Finally, configure the samba service and restart it:

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

Great! You can now configure and open the shared location inside VMs and other machines.

### Connecting to a Samba Shared Location

**Windows:** Open the "*File Explorer*" and then right-click on "*This PC*" (in the left pane). From the resulting context menu, select "*Add A Network Location*". When requested for the network address, use ``\\${IP}\files`` and continue by filling in the authentication details.

**MacOS:** Open "*Finder*" and use the menu option "*Go*" > "*Connect to Server...*". In the address field, enter ``smb://${IP}/files`` and click on "*Connect*". Accept the prompt and continue by filling in the authentication details when requested.

**Linux:** In the "*Connect to Server*" option of the file browser, enter the address as ``smb://${IP}/files`` and continue by filling in the authentication details.

**Docker:** You also can mount samba share volumes with the CIFS protocol. Check the [documentation](https://docs.docker.com/storage/volumes/#create-cifssamba-volumes) to learn how to do it.
