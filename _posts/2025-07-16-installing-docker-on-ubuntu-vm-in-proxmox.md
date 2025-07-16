---
title: "Docker on LXC... Then Pivoting to a VM"
date: 2025-07-16
categories: [lab-notes, docker, proxmox]
tags: [docker, lxc, proxmox, vm, apparmor, snapshot]
permalink: /lab-notes/proxmox/docker/lxcvsvm/
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

### Why I Chose q35 Over i440fx

When creating my Docker host VM in Proxmox, I selected q35 as the machine type. Here's why:

Modern chipset: q35 emulates newer PCI Express-based hardware, which better matches what you'd find in cloud providers and enterprise systems.

UEFI + QEMU Agent compatibility: I enabled UEFI boot and QEMU guest tools — both are fully supported by q35.

NVMe & VirtIO drivers: These work cleanly with q35, which models PCIe properly.

Future-proofing: Most newer Linux distros (Ubuntu 20.04+, Debian 11+, etc.) expect a modern virtual hardware layout.

Although i440fx is a simpler and more universally compatible option, it’s primarily helpful when installing older operating systems (e.g., Windows XP, legacy Linux).

For my modern, container-based lab environment, q35 made the most sense — and so far, it’s been 100% stable.