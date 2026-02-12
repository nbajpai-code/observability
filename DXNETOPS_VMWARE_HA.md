# Alternate Solution: DxNetOps on VMware vSphere HA

This document outlines a cost-effective and resilient architecture for deploying Broadcom DxNetOps using VMware vSphere High Availability (HA) and Fault Tolerance (FT). This approach leverages infrastructure-level resilience to complement or replace complex application-level HA.

## Architecture Overview

Instead of provisioning double the hardware for physical Active-Active application clusters (which can be costly), this solution relies on the virtualization layer to provide durability.

### Key Components
- **VMware vSphere Cluster:** A cluster of ESXi hosts managed by vCenter.
- **Shared Storage:** SAN/NAS accessible by all hosts (e.g., vSAN, iSCSI, FC) to enable vMotion and HA restart.
- **DxNetOps VMs:** Data Aggregator, Data Collector, and Performance Center deployed as Virtual Machines.
- **Vertica Database VMs:** Vertica nodes deployed as high-performance VMs.

---

## High Availability Strategy

### 1. vSphere High Availability (HA)
**Mechanism:** If a physical ESXi host fails, vSphere HA automatically restarts the affected VMs on healthy hosts within the cluster.
- **Recovery Time (RTO):** Minutes (OS boot time + Application startup time).
- **Cost Benefit:** No need for idle "standby" application licenses or hardware. Resources are pooled.
- **Use Case:** Suitable for the Portal (PC) and Data Collectors (DC) where a few minutes of downtime is acceptable.

### 2. vSphere Fault Tolerance (SMP-FT)
**Mechanism:** Creates a live shadow instance of a VM on another host. Instructions are executed in lockstep. If the primary host fails, the shadow VM takes over *instantly* with zero downtime and zero data loss.
- **Recovery Time (RTO):** Zero.
- **Cost:** Requires double the resources (CPU/RAM) for the protected VM but saves on complex application-level clustering software and management overhead.
- **Use Case:** Critical for the **Data Aggregator (DA)** to prevent polling gaps. Note: FT has vCPU limits (check your vSphere version).

---

## Vertica on VMware: Best Practices

Running a high-performance database like Vertica on VMware requires careful tuning to match bare-metal performance.

### Resource Reservations
- **Memory (RAM):** Set **100% Memory Reservation** for Vertica VMs. Ballooning or swapping kills database performance.
- **CPU:** Use **High Share** values or reservations for Vertica VMs to ensure active threads aren't de-scheduled.

### Storage & I/O
- **Paravirtual SCSI:** Use the `PVSCSI` adapter for database disks for lower CPU overhead and higher throughput.
- **Independent Disks:** Place database data/catalog drives on distinct datastores or LUNs if possible to separate I/O contention.
- **Eager Zeroed Thick Provisioning:** Use Thick Provisioned Eager Zeroed disks to prevent first-write latency.

### Affinity Rules
- **Anti-Affinity:** Create "VM-VM Anti-Affinity" rules to force Vertica cluster nodes to run on *different* physical ESXi hosts. This ensures that a single physical server failure only impacts one database node (maintaining K-safety).

---

## Cost-Benefit Analysis

| Feature | Physical Active-Active HA | VMware vSphere HA |
| :--- | :--- | :--- |
| **Hardware Cost** | High (2x full physical stacks) | Medium (Shared resource pool) |
| **Licensing** | High (Application standby licenses) | Low (Standard vSphere licensing) |
| **Complexity** | High (Dual Ingestion, Sync) | Low (Managed by vCenter) |
| **RTO (Recovery Time)** | Near Zero | Minutes (HA) / Zero (FT) |
| **Maintenance** | Complex (Upgrade 2 distinct envs) | Simple (Snapshot & Upgrade) |

## Implementation Checklist

1.  [ ] **Cluster:** Enable vSphere HA and DRS on the cluster.
2.  [ ] **Deployment:** Deploy DxNetOps components as VMs.
3.  [ ] **Protection:** Enable **Fault Tolerance** on the Data Aggregator VM (if vCPU count permits).
4.  [ ] **Optimization:** Set Latency Sensitivity to "High" for Vertica VMs.
5.  [ ] **Resilience:** Configure VM Anti-Affinity rules for all Vertica nodes.
