# Docker Containers

Besides virtual machines, you can also use the hypervisor to run containers. Follow the instructions below to install Docker:

```bash
# Traditional OS ONLY 
# Add repository and install Docker
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Immutable OS ONLY
# Install packages in OSTree layer
rpm-ostree install -y docker

# Add user permission
USERNAME="mateussouzaweb"
sudo usermod -aG docker ${USERNAME}
newgrp docker
```

Now, let's make sure Docker won't mess with bridge networks by applying some customizations on the service:

```bash
sudo systemctl edit docker.service

# Add this content to the file
[Service]
ExecStartPre=/bin/sh -c "/usr/sbin/iptables -D FORWARD -p all -i virbr0 -j ACCEPT || true"
ExecStartPre=/bin/sh -c "/usr/sbin/iptables -A FORWARD -p all -i virbr0 -j ACCEPT || true"
ExecStartPre=/bin/sh -c "/usr/sbin/ip6tables -D FORWARD -p all -i virbr0 -j ACCEPT || true"
ExecStartPre=/bin/sh -c "/usr/sbin/ip6tables -A FORWARD -p all -i virbr0 -j ACCEPT || true"
```

Finally, start the service and you are ready to run any container that you like:

```bash
sudo systemctl enable --now docker
```
