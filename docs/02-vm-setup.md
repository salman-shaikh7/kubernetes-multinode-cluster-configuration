# VM setup

Three Ubuntu VMs were created with KVM, QEMU, and libvirt and attached to libvirt's default NAT network.

| Hostname | Role | Address |
|---|---|---|
| `k8s-control-plane` | Control plane | `192.168.122.131` |
| `k8s-worker-1` | Worker | `192.168.122.94` |
| `k8s-worker-2` | Worker | `192.168.122.38` |

Configure the matching hostname on each VM:

```bash
sudo hostnamectl set-hostname k8s-control-plane
# Use k8s-worker-1 or k8s-worker-2 on the corresponding worker.
```

Add all nodes to `/etc/hosts` on every VM if local DNS does not resolve them:

```text
192.168.122.131 k8s-control-plane
192.168.122.94  k8s-worker-1
192.168.122.38  k8s-worker-2
```

Prepare every node for Kubernetes:

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
