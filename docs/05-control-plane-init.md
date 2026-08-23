# Control-plane initialization

Run on `k8s-control-plane` (`192.168.122.131`):

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.122.131 \
  --pod-network-cidr=10.244.0.0/16
```

Configure kubectl for the current user:

```bash
mkdir -p "$HOME/.kube"
sudo cp -i /etc/kubernetes/admin.conf "$HOME/.kube/config"
sudo chown "$(id -u):$(id -g)" "$HOME/.kube/config"
```

Check the control-plane components:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

The node may remain `NotReady` until Calico is installed. Continue with [Calico networking](06-calico-networking.md).

The `kubeadm init` output includes a time-limited worker join command. Do not store its token in Git. A new command can be generated later with:

```bash
sudo kubeadm token create --print-join-command
```
