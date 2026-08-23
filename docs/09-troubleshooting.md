# Troubleshooting

Start with the smallest failing layer: operating system, container runtime, kubelet, control plane, CNI, and then the workload.

## Node remains `NotReady`

```bash
kubectl describe node <node-name>
kubectl get pods -A -o wide
sudo systemctl status kubelet --no-pager
sudo journalctl -u kubelet -n 100 --no-pager
```

Common causes include an uninstalled or unhealthy CNI, swap being enabled, unreachable control-plane networking, and a stopped container runtime.

## Calico Pods are not running

```bash
kubectl get tigerastatus
kubectl get pods -n tigera-operator -o wide
kubectl get pods -n calico-system -o wide
kubectl describe pod -n calico-system <pod-name>
kubectl logs -n calico-system <pod-name> --all-containers
```

Confirm that the Calico pool and kubeadm configuration both use `10.244.0.0/16`. Also verify that `br_netfilter` is loaded and IPv4 forwarding is enabled on every node.

## Swap is still enabled

```bash
swapon --show
sudo swapoff -a
```

Remove or comment out the swap entry in `/etc/fstab`, then restart kubelet. The change must survive a reboot.

## Containerd or cgroup mismatch

```bash
systemctl is-active containerd
grep 'SystemdCgroup = true' /etc/containerd/config.toml
sudo journalctl -u containerd -n 100 --no-pager
```

If the setting is incorrect, update the configuration and restart both services:

```bash
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

## Worker join token expired

Generate a new join command on the control plane:

```bash
sudo kubeadm token create --print-join-command
```

Run the generated command on the worker. Never copy bootstrap tokens into this repository.

## Worker cannot reach the API server

From the worker, verify name resolution, routing, and TCP connectivity:

```bash
ping -c 3 192.168.122.131
nc -vz 192.168.122.131 6443
```

Check the VM network and any host or guest firewall rules if port `6443` is unreachable.

## Reset and rejoin a worker

Only run this on the worker being rebuilt:

```bash
sudo kubeadm reset
sudo systemctl restart containerd kubelet
```

Generate a fresh join command on the control plane and run it on the reset worker. Do not reset the control plane unless the intention is to rebuild the entire cluster.

## Workload Service does not respond

```bash
kubectl get pods -l app=nginx-demo -o wide
kubectl describe service nginx-demo
kubectl get endpointslice -l kubernetes.io/service-name=nginx-demo
```

The Service selector must match the Pod label, the Pods must pass their readiness probes, and the EndpointSlice must contain ready addresses.
