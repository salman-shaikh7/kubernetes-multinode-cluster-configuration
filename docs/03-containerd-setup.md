# Containerd setup

Kubernetes does not run application containers by itself. The kubelet asks a Container Runtime Interface (CRI) compatible runtime to pull images, create containers, and manage their lifecycle. This cluster uses containerd for that role.

## Install and configure the runtime

Run these steps on all three nodes. They install containerd, generate a complete default configuration, switch its cgroup driver to systemd, and make the service start automatically after a reboot.

```bash
sudo apt-get update
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Linux cgroups track and limit the CPU and memory used by processes. Kubernetes and containerd should both let systemd manage those cgroups; mismatched drivers can make nodes unstable or prevent Pods from starting.

## Verify the runtime

Confirm the setting and service state:

```bash
grep 'SystemdCgroup = true' /etc/containerd/config.toml
systemctl is-active containerd
systemctl is-enabled containerd
```

Expected results are a matching configuration line, `active`, and `enabled`. At this point containerd is ready, but no Kubernetes workloads exist yet; the kubelet will begin using it after Kubernetes is installed.
