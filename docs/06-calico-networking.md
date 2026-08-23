# Calico networking

Calico `v3.32.1` was installed with the Tigera operator. Run on the control plane:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.32.1/manifests/tigera-operator.yaml
curl -fsSLO https://raw.githubusercontent.com/projectcalico/calico/v3.32.1/manifests/custom-resources.yaml
```

Edit `custom-resources.yaml` so the installation IP pool matches the kubeadm Pod CIDR:

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

Apply it and watch Calico start:

```bash
kubectl create -f custom-resources.yaml
kubectl get tigerastatus
kubectl get pods -n calico-system -w
```

Finally, confirm that the control-plane node is ready:

```bash
kubectl get nodes -o wide
```

The CIDR must match `10.244.0.0/16`; a mismatch between kubeadm and Calico prevents correct Pod networking.
