# Observability Resources & DxNetOps HA

This repository contains a curated collection of resources for various observability domains and a high-availability design for Broadcom DxNetOps.

## Contents

### 1. [Comprehensive Observability Resources](./RESOURCES.md)
A detailed list of vendors, tools, and educational materials for:
- **Network Observability:** Broadcom, ScienceLogic, open-source tools.
- **Application Observability:** OpenTelemetry, Jaeger, APM vendors.
- **Cloud Observability:** AWS, Google Cloud, Azure.
- **LLM Observability:** LangSmith, Arize Phoenix, and more.

### 2. [DxNetOps HA & Vertica Hot Cutover](./DXNETOPS_HA.md)
An architectural guide for deploying Broadcom DxNetOps in a multi-site High Availability configuration.
- **Focus:** Minimizing failover time for the Vertica database.
- **Strategy:** Dual Ingestion (Active-Active) vs. CopyCluster.
- **Best Practices:** K-Safety, Fault Groups, and Client Failover.

### 3. [Alternate Solution: VMware vSphere HA](./DXNETOPS_VMWARE_HA.md)
A cost-effective alternative leveraging virtualization capabilities.
- **Strategy:** vSphere HA and Fault Tolerance (FT).
- **Benefit:** Reduced hardware/licensing costs with robust resilience.
- **Guide:** Best practices for virtualizing Vertica (Anti-Affinity, Resource Reservations).

## About
These resources were gathered to provide a holistic view of the current observability landscape and to offer specific architectural guidance for critical network performance management systems.
