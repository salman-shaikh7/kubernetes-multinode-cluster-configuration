# Containerd setup

Run these steps on all three nodes.

```bash
sudo apt-get update
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Kubernetes and containerd should use the same cgroup driver. Confirm the setting and service state:

```bash
grep 'SystemdCgroup = true' /etc/containerd/config.toml
systemctl is-active containerd
systemctl is-enabled containerd
```

Expected results are a matching configuration line, `active`, and `enabled`.
