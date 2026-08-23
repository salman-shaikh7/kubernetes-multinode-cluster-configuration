# Create the Ubuntu virtual machines

This guide starts on the Linux laptop and ends with three Ubuntu VMs that can reach one another and accept SSH connections.

| Hostname | Role | vCPU | RAM | Disk | Original address |
|---|---|---:|---:|---:|---|
| `k8s-control-plane` | Control plane | 2 | 4 GB | 25 GB | `192.168.122.131` |
| `k8s-worker-1` | Worker | 2 | 3 GB | 20 GB | `192.168.122.94` |
| `k8s-worker-2` | Worker | 2 | 3 GB | 20 GB | `192.168.122.38` |

The addresses above belonged to the original VMs. A new libvirt environment may assign different addresses; use those addresses throughout the remaining guide.

## 1. Install virtualization on the Linux host

Run this section on the laptop, not inside a VM. On Ubuntu or Debian:

```bash
sudo apt-get update
sudo apt-get install -y \
  qemu-kvm libvirt-daemon-system libvirt-clients \
  bridge-utils virt-manager virtinst

sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt,kvm "$USER"
```

Sign out and back in so the group change takes effect. Then validate the host:

```bash
virt-host-validate
virsh list --all
virsh net-info default
```

If the default network exists but is inactive:

```bash
sudo virsh net-start default
sudo virsh net-autostart default
```

The default network normally provides NAT, DHCP, and addresses in `192.168.122.0/24`.

## 2. Download Ubuntu Server

Download an Ubuntu Server LTS ISO from the official Ubuntu website. Ubuntu Server 24.04 LTS is a suitable choice. Keep the ISO in a known location, such as `/home/<username>/Downloads/ubuntu-server.iso`.

## 3. Create the VMs

Choose either the graphical or command-line method. Do not use both.

### Option A: virt-manager

Open Virtual Machine Manager:

```bash
virt-manager
```

For each node:

1. Select **Create a new virtual machine**.
2. Choose **Local install media (ISO image or CDROM)**.
3. Select the downloaded Ubuntu Server ISO.
4. Assign the CPU, memory, and disk from the table above.
5. Set the VM name to `k8s-control-plane`, `k8s-worker-1`, or `k8s-worker-2`.
6. Confirm the network source is **Virtual network 'default': NAT** and the device model is `virtio`.
7. Start the VM and complete the Ubuntu installation.

During installation, create an administrative user, select the OpenSSH server if offered, and give each VM its matching unique hostname. Detach the ISO afterward if the VM starts the installer again.

### Option B: virt-install

Replace `/path/to/ubuntu-server.iso` with the real absolute ISO path:

```bash
sudo virt-install \
  --name k8s-control-plane --vcpus 2 --memory 4096 \
  --disk size=25,bus=virtio \
  --cdrom /path/to/ubuntu-server.iso \
  --network network=default,model=virtio --graphics spice

sudo virt-install \
  --name k8s-worker-1 --vcpus 2 --memory 3072 \
  --disk size=20,bus=virtio \
  --cdrom /path/to/ubuntu-server.iso \
  --network network=default,model=virtio --graphics spice

sudo virt-install \
  --name k8s-worker-2 --vcpus 2 --memory 3072 \
  --disk size=20,bus=virtio \
  --cdrom /path/to/ubuntu-server.iso \
  --network network=default,model=virtio --graphics spice
```

Each command opens the Ubuntu installer. Complete one installation before moving to the next.

## 4. Configure each Ubuntu VM

Use the VM console initially. Run only the hostname command belonging to the current VM:

```bash
sudo hostnamectl set-hostname k8s-control-plane
sudo hostnamectl set-hostname k8s-worker-1
sudo hostnamectl set-hostname k8s-worker-2
```

Install SSH and the QEMU guest agent:

```bash
sudo apt-get update
sudo apt-get install -y openssh-server qemu-guest-agent
sudo systemctl enable --now ssh qemu-guest-agent
```

If Ubuntu's firewall is active, allow SSH:

```bash
sudo ufw status
sudo ufw allow OpenSSH
```

Verify inside each VM:

```bash
hostname
hostname -I
systemctl is-active ssh
ip -brief address
```

## 5. Find each VM address

On the Linux host:

```bash
virsh list --all
virsh net-dhcp-leases default
```

With the guest agent running, you can query each VM directly:

```bash
virsh domifaddr k8s-control-plane --source agent
virsh domifaddr k8s-worker-1 --source agent
virsh domifaddr k8s-worker-2 --source agent
```

Ignore the loopback address and record the address on the libvirt network.

## 6. Connect over SSH

From the Linux host, replace `<ubuntu-user>` with the user created during installation:

```bash
ssh <ubuntu-user>@192.168.122.131
ssh <ubuntu-user>@192.168.122.94
ssh <ubuntu-user>@192.168.122.38
```

On the first connection, verify the displayed host, accept its host key, and enter the VM user's password.

### Optional: use SSH keys

Create a key on the Linux host if one does not already exist:

```bash
ssh-keygen -t ed25519 -C "k8s-lab"
```

Copy it to each VM:

```bash
ssh-copy-id <ubuntu-user>@192.168.122.131
ssh-copy-id <ubuntu-user>@192.168.122.94
ssh-copy-id <ubuntu-user>@192.168.122.38
```

### Optional: create SSH aliases

Add this to `~/.ssh/config` on the Linux host, replacing `<ubuntu-user>`:

```sshconfig
Host k8s-control-plane
    HostName 192.168.122.131
    User <ubuntu-user>

Host k8s-worker-1
    HostName 192.168.122.94
    User <ubuntu-user>

Host k8s-worker-2
    HostName 192.168.122.38
    User <ubuntu-user>
```

You can now connect with `ssh k8s-control-plane`, `ssh k8s-worker-1`, or `ssh k8s-worker-2`.

## 7. Keep the addresses stable

Libvirt DHCP addresses can change. Create DHCP reservations using each VM's MAC address:

```bash
virsh domiflist k8s-control-plane
virsh domiflist k8s-worker-1
virsh domiflist k8s-worker-2
sudo virsh net-edit default
```

Inside the existing `<dhcp>` section, add reservations with the real MAC addresses:

```xml
<host mac='CONTROL_PLANE_MAC' name='k8s-control-plane' ip='192.168.122.131'/>
<host mac='WORKER_1_MAC' name='k8s-worker-1' ip='192.168.122.94'/>
<host mac='WORKER_2_MAC' name='k8s-worker-2' ip='192.168.122.38'/>
```

Restart the VMs afterward. If an address is already assigned to another machine, choose an unused address and update the remaining documentation accordingly.

## 8. Configure and test name resolution

Add the final mapping to `/etc/hosts` on all three VMs:

```text
192.168.122.131 k8s-control-plane
192.168.122.94  k8s-worker-1
192.168.122.38  k8s-worker-2
```

You may add the same entries to the Linux host. Verify from every VM:

```bash
getent hosts k8s-control-plane k8s-worker-1 k8s-worker-2
ping -c 2 k8s-control-plane
ping -c 2 k8s-worker-1
ping -c 2 k8s-worker-2
```

Port `6443` will not open until kubeadm initializes the control plane, but workers should already be able to route to `192.168.122.131`.

## 9. Prepare every node for Kubernetes

Run on all three VMs:

```bash
sudo swapoff -a
# Also comment out any swap entry in /etc/fstab so swap remains disabled after reboot.

cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<'EOF' | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

Validate:

```bash
swapon --show
lsmod | grep -E 'overlay|br_netfilter'
sysctl net.bridge.bridge-nf-call-iptables net.ipv4.ip_forward
```

`swapon --show` should return no active swap devices, and both sysctl values should be `1`.

The VMs are now ready for [containerd installation](03-containerd-setup.md).
