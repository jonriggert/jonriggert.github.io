---
title: "Ticket #001: Docker on LXC... Then Pivoting to a VM"
date: 2025-07-16
categories: [lab-notes, docker, proxmox]
tags: [docker, lxc, proxmox, vm, apparmor, snapshot]
---

## Install Docker in Proxmox LXC... The Pivoting to an Ubuntu VM

My goal was to install Docker inside an LXC container to keep it lightweight and fast. I created a privileged LXC container (`docker-host-lxc01`) in Proxmox using the `ubuntu-22.04-standard` template, with 2 vCPU, 2GB RAM, and 20GB disk.

I used the official Docker install method:

```bash
curl -fsSL https://get.docker.com | sudo sh
```

But after the install, docker run hello-world kept failing with this error:

```bash
AppArmor enabled on system but the docker-default profile could not be loaded...
```

Despite setting lxc.apparmor.profile: unconfined and enabling nesting, AppArmor kept blocking Docker from starting containers. I tried:

- Manually setting DNS

- Editing the LXC config

- Restarting and resetting AppArmor

- Using pct set commands to adjust permissions

No luck. Eventually, I decided to pivot to a VM for better stability.

### Create Docker Host Ubuntu VM
I created a clean VM called docker-host-vm01 using the q35 chipset and the Ubuntu 22.04 ISO.

VM Specs:
- 2 vCPU

- 2GB RAM

- 20GB VirtIO disk

- UEFI boot + QEMU Agent

- Network: vmbr0, VirtIO model

After installing Ubuntu (minimal), I ran:

```bash
Copy
Edit
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
newgrp docker
```

Tested it with:

```bash
Copy
Edit
docker run hello-world
```

Success! Docker printed the welcome message. Internet access and DNS also worked perfectly.

Snapshot Saved
Before moving forward, I took a Proxmox snapshot of the working Docker VM so I can reuse it later.