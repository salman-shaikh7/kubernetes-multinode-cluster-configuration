# Prerequisites

## Host machine

This lab assumes a Linux host with hardware virtualization enabled in its BIOS or UEFI settings.

Recommended host capacity:

| Resource | Recommended |
|---|---|
| CPU | 8 logical cores or more |
| Memory | 16 GB RAM or more |
| Free storage | 80 GB or more |
| Network | Internet access during installation |

The host must have KVM, QEMU, libvirt, and a VM management tool such as `virt-manager` or `virt-install`. Confirm that virtualization is available:

```bash
lscpu | grep Virtualization
virsh list --all
virsh net-info default
```

## Virtual machines

Use three Ubuntu Server VMs. Ubuntu 24.04 LTS is a suitable choice; if a different release is used, package names or defaults may vary.

| Node | vCPU | RAM | Storage |
|---|---:|---:|---:|
| Control plane | 2 | 4 GB | 25 GB |
| Worker 1 | 2 | 3 GB | 20 GB |
| Worker 2 | 2 | 3 GB | 20 GB |

These values are appropriate for a learning lab and small demo workloads. Airflow, Spark, or other future platforms may require additional CPU and memory.

Each VM needs:

- A unique hostname and MAC address
- An address reachable by every other VM
- Internet access for Ubuntu, Kubernetes, containerd, Calico, and container images
- SSH access for convenient administration
- Consistent system time

This repository records the addresses used by the original cluster. If DHCP assigns different addresses after a restart, update the configuration or create stable libvirt DHCP reservations.

## Required access

The setup commands require a user with `sudo` privileges. Run cluster administration commands from the control-plane VM after configuring `kubectl`.

## Version note

This guide records the tested lab configuration: Kubernetes `v1.34.11` and Calico `v3.32.1`. Package repositories, installation URLs, and compatibility requirements can change. Keep versions pinned while reproducing the lab, and review upstream upgrade documentation before changing them.
