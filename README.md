# Kubernetes Multi-Node Cluster Configuration

A reproducible three-node Kubernetes lab built on a Linux laptop with KVM, QEMU, libvirt, Ubuntu virtual machines, containerd, kubeadm, and Calico.

## Cluster at a glance

| Role | Hostname | libvirt NAT address |
|---|---|---|
| Control plane | `k8s-control-plane` | `192.168.122.131` |
| Worker | `k8s-worker-1` | `192.168.122.94` |
| Worker | `k8s-worker-2` | `192.168.122.38` |

| Component | Configuration |
|---|---|
| Kubernetes | `v1.34.11` |
| Container runtime | containerd, `SystemdCgroup = true` |
| Pod network | `10.244.0.0/16` |
| CNI | Calico `v3.32.1`, operator installation |
| Virtualization | KVM + QEMU + libvirt |
| Guest OS | Ubuntu |

```text
Linux laptop
└── KVM / QEMU / libvirt
    ├── k8s-control-plane (192.168.122.131)
    ├── k8s-worker-1      (192.168.122.94)
    └── k8s-worker-2      (192.168.122.38)
         └── containerd → Kubernetes → Calico
```

## What was completed

- Created one control-plane VM and two worker VMs on libvirt's NAT network.
- Disabled swap and configured the required kernel modules and forwarding settings on every node.
- Installed containerd and enabled its systemd cgroup driver.
- Installed Kubernetes `v1.34.11` and initialized the control plane with kubeadm.
- Installed Calico `v3.32.1` with a Pod CIDR of `10.244.0.0/16`.
- Joined both workers and verified that all three nodes became `Ready`.
- Deployed a three-replica nginx workload, exposed it through a ClusterIP Service, and verified service discovery and load distribution.

## Guide

Follow these documents in order:

1. [Architecture](docs/01-architecture.md)
2. [VM setup](docs/02-vm-setup.md)
3. [Containerd setup](docs/03-containerd-setup.md)
4. [Kubernetes installation](docs/04-kubernetes-installation.md)
5. [Control-plane initialization](docs/05-control-plane-init.md)
6. [Calico networking](docs/06-calico-networking.md)
7. [Worker join](docs/07-worker-join.md)
8. [Service and load-balancing test](docs/08-service-load-balancing.md)

## Quick validation

Run on the control plane:

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get svc,endpointslice
```

Expected node state:

```text
k8s-control-plane   Ready   control-plane
k8s-worker-1        Ready   <none>
k8s-worker-2        Ready   <none>
```

## Demo workload

```bash
kubectl apply -f manifests/nginx-demo.yaml
kubectl get deployment,pods,service,endpointslice -l app=nginx-demo
```

The reusable manifest creates a three-replica nginx Deployment and a ClusterIP Service named `nginx-demo`.

## Security note

Kubeadm bootstrap tokens and certificate hashes expire and should not be committed. Generate a fresh worker join command when needed:

```bash
kubeadm token create --print-join-command
```

## Cleanup

```bash
kubectl delete -f manifests/nginx-demo.yaml
```
