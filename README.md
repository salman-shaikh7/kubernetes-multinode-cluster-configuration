# Kubernetes Multi-Node Cluster Configuration

A reproducible three-node Kubernetes lab built on a Linux laptop with KVM, QEMU, libvirt, Ubuntu virtual machines, containerd, kubeadm, and Calico.

## Motivation

Cloud providers make Kubernetes easy to provision, but that convenience comes with many layers of abstraction. A managed service can create the control plane, networking, worker nodes, and supporting infrastructure automatically. That is valuable in production, but it can hide how the individual components are installed, configured, connected, and troubleshot.

I built this cluster locally to learn Kubernetes from the ground up. Running it on my own virtual machines gives me full control over the operating system, container runtime, networking, control plane, and worker-node configuration. I can inspect every layer, break and rebuild the cluster, and understand what Kubernetes is doing behind the abstractions provided by managed cloud services. It also gives me a reusable learning environment without ongoing cloud infrastructure costs.

### Why not Minikube or another single-node setup?

Tools such as Minikube are excellent for learning Kubernetes commands, deploying applications, and experimenting quickly. However, my goal is to understand how a realistic multi-node cluster is assembled and operated. A control plane with separate worker nodes more closely simulates a real environment and exposes important concepts such as node preparation, cluster bootstrapping, worker joins, cross-node Pod networking, service discovery, scheduling, and failure investigation.

### Foundation for future projects

This cluster is not the final project; it is the foundation for future distributed-systems projects. I plan to use it to build and study platforms such as Apache Airflow and Apache Spark, along with other clustered applications. Building those systems on infrastructure I control will help me understand how their components communicate, scale, recover, and behave across multiple machines.

For continuous hands-on experimentation, relying entirely on managed cloud services would add cost and make low-level exploration more difficult. This local lab provides a cost-effective and repeatable environment where I can learn each system from first principles, while still developing skills that transfer to cloud-based Kubernetes environments.

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

1. [Prerequisites](docs/00-prerequisites.md)
2. [Architecture](docs/01-architecture.md)
3. [VM creation, networking, and SSH](docs/02-vm-setup.md)
4. [Containerd setup](docs/03-containerd-setup.md)
5. [Kubernetes installation](docs/04-kubernetes-installation.md)
6. [Control-plane initialization](docs/05-control-plane-init.md)
7. [Calico networking](docs/06-calico-networking.md)
8. [Worker join](docs/07-worker-join.md)
9. [Service and load-balancing test](docs/08-service-load-balancing.md)
10. [Troubleshooting](docs/09-troubleshooting.md)
11. [Validation evidence](docs/10-validation-evidence.md)

## Repository contents

- `docs/` — the complete build, test, and troubleshooting guide
- `manifests/calico-custom-resources.yaml` — the pinned Calico network configuration
- `manifests/nginx-demo.yaml` — a reusable three-replica nginx Deployment and ClusterIP Service
- `assets/screenshots/` — sanitized validation evidence from the running cluster

## License

This project is available under the [MIT License](LICENSE).
