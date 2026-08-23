# Calico networking

Kubernetes defines how Pods should be networked but does not implement that network itself. A Container Network Interface (CNI) plugin assigns Pod addresses and carries traffic between Pods on the same or different nodes. This cluster uses Calico `v3.32.1`.

## Install the operator and network configuration

Run on the control plane:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.32.1/manifests/tigera-operator.yaml
kubectl create -f manifests/calico-custom-resources.yaml
```

The first manifest installs the Tigera operator, which manages the lifecycle of Calico components. The second describes the network this cluster needs. Keeping that custom resource in the repository makes the chosen network reproducible.

The IP pool matches the Pod CIDR passed to kubeadm:

```yaml
spec:
  calicoNetwork:
    ipPools:
      - blockSize: 26
        cidr: 10.244.0.0/16
        encapsulation: VXLANCrossSubnet
        natOutgoing: Enabled
        nodeSelector: all()
```

`VXLANCrossSubnet` encapsulates cross-subnet Pod traffic, while `natOutgoing: Enabled` allows Pods to reach destinations outside the cluster through a node address.

## Wait for the network to become available

`tigerastatus` summarizes the operator-managed components. The Pod listing shows the Calico agents and controllers starting:

```bash
kubectl get tigerastatus
kubectl get pods -n calico-system -w
```

Finally, confirm that the control-plane node is ready. Becoming `Ready` means the kubelet now has a functioning Pod network:

```bash
kubectl get nodes -o wide
```

The CIDR must match `10.244.0.0/16`; a mismatch between kubeadm and Calico prevents correct Pod networking.
