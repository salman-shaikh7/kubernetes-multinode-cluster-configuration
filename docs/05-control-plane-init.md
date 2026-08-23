# Control-plane initialization

`kubeadm init` turns the prepared VM into a control-plane node. It generates cluster certificates, creates kubeconfig files, writes the static Pod definitions for the API server, scheduler, controller manager, and etcd, and starts those components through the kubelet.

## Bootstrap the control plane

Run on `k8s-control-plane` (`192.168.122.131`):

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.122.131 \
  --pod-network-cidr=10.244.0.0/16
```

`--apiserver-advertise-address` tells other nodes how to reach the Kubernetes API. `--pod-network-cidr` reserves an address range for Pods; it must match the range later configured in Calico.

## Give the current user kubectl access

The generated `admin.conf` contains the cluster address and administrator credentials. Copy it into the current user's kubeconfig location so `kubectl` can authenticate without running as root:

```bash
mkdir -p "$HOME/.kube"
sudo cp -i /etc/kubernetes/admin.conf "$HOME/.kube/config"
sudo chown "$(id -u):$(id -g)" "$HOME/.kube/config"
```

Treat `$HOME/.kube/config` as a credential: do not commit it or share its contents.

## Check the initial state

These commands confirm that the API responds and show the system Pods created during initialization:

```bash
kubectl get nodes
kubectl get pods -n kube-system
```

The node may remain `NotReady` because Kubernetes does not yet have a Container Network Interface plugin. Continue with [Calico networking](06-calico-networking.md).

The `kubeadm init` output also includes a time-limited worker join command. The token proves that a joining machine is authorized, while the CA hash lets it verify the identity of the control plane. Do not store that command in Git. Generate a new one later when needed:

```bash
sudo kubeadm token create --print-join-command
```
