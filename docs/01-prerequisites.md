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
To provision these four VMs on an Ubuntu host using KVM/QEMU, the most efficient method is using `virt-install`. This assumes you have the Alpine Linux "Virtual" ISO downloaded.

### 1. Prerequisites

Install the necessary virtualization packages and start the default network (which provides a NAT bridge for VM-to-VM communication).

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients virtinst bridge-utils

# Ensure the default network is active
sudo virsh net-start default
sudo virsh net-autostart default

```

---

### 2. Prepare the Storage

Create a directory to house your virtual disk images.

```bash
sudo mkdir -p /var/lib/libvirt/images/alpine-cluster
cd /var/lib/libvirt/images/alpine-cluster

```

---

### 3. Provisioning Commands

Run these commands to create each VM. Replace `/path/to/alpine-virt.iso` with the actual path to your downloaded Alpine ISO.

#### Jumpbox

```bash
virt-install \
  --name jumpbox \
  --ram 512 \
  --vcpus 1 \
  --disk size=10,path=/var/lib/libvirt/images/alpine-cluster/jumpbox.qcow2 \
  --os-variant alpine3.18 \
  --network network=default \
  --graphics none \
  --console pty,target_type=serial \
  --location /path/to/alpine-virt.iso \
  --extra-args "console=ttyS0"

```

#### Kubernetes Server

```bash
virt-install \
  --name server \
  --ram 2048 \
  --vcpus 1 \
  --disk size=20,path=/var/lib/libvirt/images/alpine-cluster/server.qcow2 \
  --os-variant alpine3.18 \
  --network network=default \
  --graphics none \
  --console pty,target_type=serial \
  --location /path/to/alpine-virt.iso \
  --extra-args "console=ttyS0"

```

#### Worker Node 0

```bash
virt-install \
  --name node-0 \
  --ram 2048 \
  --vcpus 1 \
  --disk size=20,path=/var/lib/libvirt/images/alpine-cluster/node-0.qcow2 \
  --os-variant alpine3.18 \
  --network network=default \
  --graphics none \
  --console pty,target_type=serial \
  --location /path/to/alpine-virt.iso \
  --extra-args "console=ttyS0"

```

#### Worker Node 1

```bash
virt-install \
  --name node-1 \
  --ram 2048 \
  --vcpus 1 \
  --disk size=20,path=/var/lib/libvirt/images/alpine-cluster/node-1.qcow2 \
  --os-variant alpine3.18 \
  --network network=default \
  --graphics none \
  --console pty,target_type=serial \
  --location /path/to/alpine-virt.iso \
  --extra-args "console=ttyS0"

```

---

### 4. Networking and Connectivity

By using `--network network=default`, all four VMs are attached to the `virbr0` bridge. They will receive IP addresses via DHCP in the `192.168.122.x` range and can communicate with each other immediately.

To find their IP addresses once they are booted:

```bash
virsh net-dhcp-leases default

```

### 5. Post-Installation Note

Once the `virt-install` command triggers the boot, you will be inside the Alpine console. You must run the Alpine setup script to install the OS to the virtual disk, as Alpine runs in RAM by default:

```bash
setup-alpine

```

Select `sys` as the disk mode to ensure the OS is installed permanently to the `.qcow2` files.
'''

Once you have all four machines provisioned, verify the OS requirements by viewing the `/etc/os-release` file:

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
