---
theme: gaia
_class: lead
paginate: true
size: 16:9
style: |
  section {
    font-size: 24px;
  }
  h1 {
    font-size: 40px;
  }
  h2 {
    font-size: 30px;
  }
  table {
    font-size: 20px;
  }
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

# DxNetOps High Availability
## A Cost-Effective VMware vSphere Solution

**Broadcom DxNetOps + VMware vSphere**

---

# Executive Summary

**Objective:** Achieve High Availability (HA) and Disaster Recovery (DR) for Broadcom DxNetOps without the prohibitive cost of duplicate physical hardware.

**Solution:** Leverage **VMware vSphere HA** and **Fault Tolerance (FT)** to provide infrastructure-level resilience.

**Outcome:**
- Reduced Hardware Costs
- Simplified Management
- Near-Zero RTO for Critical Components

---

# The Challenge: Traditional Physical HA

Deploying DxNetOps on bare metal requires:
- **Double Hardware:** 1-to-1 mapping for Active/Standby clusters.
- **Complex Licensing:** Idle standby licenses often required.
- **Manual Failover:** Physical failover can be complex and slow.
- **High TCO:** Significant CapEx and OpEx for minimal resilience gain.

---

# The Solution: Virtualized Resilience

By virtualizing DxNetOps on VMware vSphere, we shift resilience from the *Application* layer to the *Infrastructure* layer.

### Key Technologies
1.  **vSphere HA:** Automatic restart of VMs on healthy hosts.
2.  **vSphere Fault Tolerance (SMP-FT):** Instant failover with zero data loss.
3.  **vMotion:** Zero-downtime maintenance.

![h:400 center](./images/dxnetops_architecture_hightech.png)

---

# Visualizing Fault Tolerance

![h:350 center](./images/fault_tolerance_hightech.png)

*Comparison of active execution path between Primary and Shadow VMs.*

---

# Deep Dive: Component Strategy

| **App** | **Strategy** | **RTO** | **Rationale** |
| :--- | :--- | :--- | :--- |
| **Data Aggregator** | **Fault Tolerance** | **Zero** | Critical; no polling gaps. |
| **Performance Ctr** | **vSphere HA** | Mins | UI critical; tolerates reboot. |
| **Data Collectors** | **vSphere HA** | Mins | Buffers data during outage. |
| **Vertica DB** | **HA + Anti-Affinity** | Mins | Priority: K-safety & consistency. |

---

# Vertica on VMware: Best Practices

To ensure bare-metal performance for the Data Repository:

*   **100% Memory Reservation:** Prevent swapping/ballooning.
*   **Paravirtual SCSI (PVSCSI):** Low CPU overhead for high I/O.
*   **Thick Provision Eager Zeroed:** Eliminate first-write latency.

![bg right:40% fit](./images/anti_affinity_hightech.png)
*   **Anti-Affinity Rules:** Ensure Vertica nodes *never* share a physical host.

---

# Cost-Benefit Analysis

| Feature | Physical Active-Active | VMware vSphere Solution |
| :--- | :--- | :--- |
| **CapEx** | $$$$ (2x Hardware) | $$ (Shared Pool) |
| **OpEx** | $$$$ (Power/Cooling) | $$ (Consolidated) |
| **Licensing** | Double (Standby) | Standard vSphere |
| **Complexity** | High (Custom Scripts) | Low (Native) |

---

# Conclusion

Adopting a **VMware vSphere-based HA architecture** for Broadcom DxNetOps offers the optimal balance of:

1.  **Resilience:** Enterprise-grade protection (up to 99.99%).
2.  **Simplicity:** No complex application clustering to manage.
3.  **Cost Efficiency:** Maximizes hardware utilization.

**Recommendation:** Proceed with virtualized deployment using Fault Tolerance for the Data Aggregator and HA for all other components.
