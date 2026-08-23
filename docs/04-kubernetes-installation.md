# Kubernetes installation

Kubernetes `v1.34.11` was installed from the official `v1.34` package repository. Run the following on all three nodes:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

Inspect the available package version, then install the `1.34.11` package shown by apt:

```bash
apt-cache madison kubeadm | grep 1.34.11
K8S_PACKAGE_VERSION="$(apt-cache madison kubeadm | awk '$3 ~ /^1\.34\.11-/{print $3; exit}')"
test -n "$K8S_PACKAGE_VERSION"
sudo apt-get install -y \
  kubelet="$K8S_PACKAGE_VERSION" \
  kubeadm="$K8S_PACKAGE_VERSION" \
  kubectl="$K8S_PACKAGE_VERSION"
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable kubelet
```

Only `kubeadm` and `kubelet` are required on worker nodes; installing `kubectl` there is optional. The original lab used all three tools on the control plane and `kubeadm` plus `kubelet` on workers.

Verify:

```bash
kubeadm version -o short
kubelet --version
kubectl version --client
```
