# Prerequisites

In this lab you will review the machine requirements necessary to follow this tutorial.

## Virtual or Physical Machines

* This tutorial requires four (4) virtual or physical ARM64 or AMD64 machines running Alpine Linux. 
* The following table lists the four machines and their CPU, memory, and storage requirements.

| Name    | Description            | CPU | RAM   | Storage |
|---------|------------------------|-----|-------|---------|
| jumpbox | Administration host    | 1   | 512MB | 10GB    |
| server  | Kubernetes server      | 1   | 2GB   | 20GB    |
| node-0  | Kubernetes worker node | 1   | 2GB   | 20GB    |
| node-1  | Kubernetes worker node | 1   | 2GB   | 20GB    |
| TOTAL   |  total resources       | 4   | 6.5GB | 70GB    | 

## Provision the four VMs using QEMU + KVM 

* download the alpine linux ISO

'''
To provision Alpine Linux non-interactively using KVM/QEMU, the standard approach is to use a **Cloud Image** combined with **Cloud-Init**. Unlike the "Virtual" ISO which requires a manual install script, the Cloud Image is a pre-installed disk template that configures itself on the first boot.

### 1. Download the Alpine Cloud Image

On your Ubuntu host, download the latest Alpine "cloud" image (standard qcow2 format).

```bash
mkdir -p /var/lib/libvirt/images/alpine-cluster
cd /var/lib/libvirt/images/alpine-cluster

# Download the x86_64 cloud image (Update version number if a newer one is available)
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/cloud/nocloud_alpine-3.20.3-x86_64-bios-cloudinit-r0.qcow2 -O alpine-base.qcow2

```

---

### 2. Create the Cloud-Init Config

Cloud-init uses a `user-data` file to set the password, SSH keys, and run initial commands (like installing Kubernetes dependencies).

**Create a file named `user-data`:**

```yaml
#cloud-config
password: "your-safe-password"
chpasswd: { expire: False }
ssh_pwauth: True
ssh_authorized_keys:
  - ssh-ed25519 AAAAC3Nza... your_email@example.com

# Automatically update and install basic tools
apk:
  repositories:
    - https://dl-cdn.alpinelinux.org/alpine/v3.20/main
    - https://dl-cdn.alpinelinux.org/alpine/v3.20/community
  packages:
    - curl
    - bash
    - htop

runcmd:
  - setup-hostname ${hostname}

```

---

### 3. Provisioning Script

Because we are using a template, we need to create a unique copy of the base disk for each VM. The following script automates the disk creation and the `virt-install` process for all four VMs simultaneously.

```bash
#!/bin/bash

# Define resources in an array: Name|RAM|DiskSize
VMS=(
  "jumpbox|512|10"
  "server|2048|20"
  "node-0|2048|20"
  "node-1|2048|20"
)

BASE_IMG="/var/lib/libvirt/images/alpine-cluster/alpine-base.qcow2"

for vm in "${VMS[@]}"; do
  IFS='|' read -r NAME RAM DISK <<< "$vm"
  DISK_PATH="/var/lib/libvirt/images/alpine-cluster/${NAME}.qcow2"

  # 1. Create a linked clone (saves space) or a full copy
  # Using a copy here to match your specific disk size requirements
  cp "$BASE_IMG" "$DISK_PATH"
  qemu-img resize "$DISK_PATH" "${DISK}G"

  # 2. Provision with cloud-init
  virt-install \
    --name "$NAME" \
    --ram "$RAM" \
    --vcpus 1 \
    --disk path="$DISK_PATH",format=qcow2 \
    --os-variant alpinelinux3.18 \
    --network network=default \
    --cloud-init user-data=user-data \
    --graphics none \
    --noautoconsole \
    --import
done

```

### 4. Verification

Once the script finishes, the VMs will boot in the background. They will automatically resize their partitions and apply your settings.

* **Check Status:** `virsh list --all`
* **Check IP Addresses:** `virsh net-dhcp-leases default`
* **Access a VM:** `virsh console jumpbox` (Use `Ctrl + ]` to exit the console)

By using `--import`, `virt-install` skips the OS installation phase entirely because the disk image is already "ready to run." All customization is handled by the `user-data` file provided via the `--cloud-init` flag.
'''

### CHECK THE OS 

```bash
cat /etc/os-release
```

You should see something similar to the following output:

```text
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.21.5
PRETTY_NAME="Alpine Linux v3.21"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
```

Next: [setting-up-the-jumpbox](02-jumpbox.md)
