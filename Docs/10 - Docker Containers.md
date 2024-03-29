# Docker Containers

Besides virtual machines, you can also use the hypervisor to run containers. Follow the instructions below to install Docker:

```bash
# Add repository
dnf -y install dnf-plugins-core
dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo

# Install docker
dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Add user permission
USERNAME="mateussouzaweb"
sudo usermod -aG docker ${USERNAME}
newgrp docker
```

Now, let's make sure Docker won't mess with bridge network by applying some customizations on the service:

```bash
sudo systemctl edit docker.service

# Add this content to the file
[Service]
ExecStartPre=/bin/sh -c "/usr/sbin/iptables -D FORWARD -p all -i virbr0 -j ACCEPT || true"
ExecStartPre=/bin/sh -c "/usr/sbin/iptables -A FORWARD -p all -i virbr0 -j ACCEPT || true"
```

Finally, start the service and you are ready to run any container that you like:

```bash
systemctl enable --now docker
```