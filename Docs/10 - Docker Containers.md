# Docker Containers

Besides virtual machines, you can also use the hypervisor to run containers. Follow the instructions below to install Docker:

![Fedora](../Images/fedora.png)
**FEDORA - DESKTOP / SERVER:**

```bash
# Fedora ONLY
# Add repository and install Docker
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

![Fedora](../Images/fedora.png)
**FEDORA - ATOMIC:**

```bash
# Fedora Immutable ONLY
# Install packages in OSTree layer
rpm-ostree install -y docker
```

![Ubuntu](../Images/ubuntu.png)
**UBUNTU - DESKTOP / SERVER:**

```bash
# Ubuntu ONLY
# Download and install docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

To run docker container without issues, your user need permissions to manipulate containers. You can add your user to the docker group by running the following commands:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
# Add user permission
USERNAME="mateussouzaweb"
sudo usermod -aG docker ${USERNAME}
newgrp docker
```

Now, let's make sure Docker won't mess with bridge networks by applying some customizations on the service:

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

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

![Fedora](../Images/fedora.png)
![Ubuntu](../Images/ubuntu.png)
**FEDORA / UBUNTU:**

```bash
sudo systemctl enable --now docker
```
