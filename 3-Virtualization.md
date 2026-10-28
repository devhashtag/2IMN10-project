# Virtualization

> Virtualization is partitioning a set of physical resources into multiple isolated sets of logical resources

Depending on type, it increases isolation and portability

RAM -> virtual memory's
Core -> Virtual cpus
machine -> virtual machines
LAN -> VLANs

## Motivation
1. Resource sharing -> pay per usage, for cloud providers, SLA determines quality
2. Isolation: VM has better isolation than a process. Performance isolation can be an issue
3. Portability. Virtualized resources are assigned to resources managers with an API. (e.g. proxmox node, easy migration)

## Types
Type 1 runs hypervisor in server, and then full VMs on top of it.

Type 2 runs a Host OS between server and huypervisor.

Type 3 is type 2 but the VM's have no guest OS (so no true VM). The use the host kernel instead, and have their own libraries. usually meant for running a single service. They are visible as processes in the Host OS. (Docker)


- VM has big resources footprint, container small
- VM has large start up time, container small,
- VM has full memory isolation, containers do not
