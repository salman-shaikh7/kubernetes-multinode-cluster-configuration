# Calico networking

Calico `v3.32.1` was installed with the Tigera operator. Run on the control plane:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.32.1/manifests/tigera-operator.yaml
kubectl create -f manifests/calico-custom-resources.yaml
```

The version-controlled custom resource sets the installation IP pool to match the kubeadm Pod CIDR:

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

Watch Calico start:

```bash
kubectl get tigerastatus
kubectl get pods -n calico-system -w
```

Finally, confirm that the control-plane node is ready:

```bash
kubectl get nodes -o wide
```

The CIDR must match `10.244.0.0/16`; a mismatch between kubeadm and Calico prevents correct Pod networking.
