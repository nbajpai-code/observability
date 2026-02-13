# Cisco DNAC Integration with Broadcom DxNetOps VNA: A Deep Dive

## Executive Summary
This document provides a comprehensive guide to integrating **Cisco Digital Network Architecture Center (DNAC)** with **Broadcom DxNetOps Virtual Network Assurance (VNA)**. By leveraging the VNA plugin, network operations teams can pull rich telemetry, inventory, and health metrics from their SD-Access and traditional Cisco environments directly into the DxNetOps Portal. This unification is critical for end-to-end visibility, enabling rapid triage of application responsiveness issues and network performance degradation.

## 1. Architecture & Prerequisites

The integration relies on the **VNA Gateway**, which acts as a mediation layer collecting data from the SDN controller (Cisco DNAC) and normalizing it for the **DxNetOps Performance Management (PM)** and **Spectrum** platforms.

### Components
*   **Cisco DNA Center**: Typical versions supported include **2.2.3.x** and **2.3.5.x**. It serves as the source of truth for the SD-Access fabric.
*   **DxNetOps VNA**: The collector that polls DNAC API endpoints.
*   **DxNetOps Portal (NetOps Portal)**: The single pane of glass for visualization.

### Prerequisites
*   IP connectivity between the VNA server and the Cisco DNAC cluster (HTTPS/443).
*   A user account on Cisco DNAC with `ROLE_OBSERVER` (Northbound API access).
*   DxNetOps VNA installed and running.

---

## 2. Configuration Guide: VNA Plugin

The VNA plugin for Cisco DNAC is configured via a JSON template. You can apply this configuration through the VNA REST API or the DxNetOps Portal UI.

### Step-by-Step Configuration
1.  **Access VNA**: Log in to the VNA Gateway UI or prepare to use `curl`.
2.  **Prepare JSON**: Modify the standard template with your specific DNAC credentials and polling intervals.
3.  **Deploy**: POST the configuration to the VNA engine.

### Sample JSON Configuration
Below is a production-ready JSON template for the Cisco DNAC plugin.

```json
{
  "pluginName": "CiscoDNA",
  "pluginType": "CISCO_DNA_CENTER",
  "description": "Production DNAC Integration for SD-Access Fabric",
  "adminStatus": "ENABLED",
  "pollingInterval": 5,
  "config": {
    "hosts": [
      {
        "hostName": "dnac-cluster-vip.example.com",
        "port": 443,
        "protocol": "https",
        "username": "admin_observer",
        "password": "YOUR_SECURE_PASSWORD",
        "proxy": {
            "enabled": false
        },
        "domain": "Default Domain"
      }
    ],
    "inventoryOptions": {
        "includeSites": true,
        "includeDevices": true,
        "includeInterfaces": true,
        "includeAccessPoints": true,
        "includeWirelessControllers": true
    },
    "assuranceOptions": {
        "collectHealthScores": true,
        "collectClientStats": true,
        "collectAppHealth": true
    }
  }
}
```

> [!IMPORTANT]
> Ensure `pollingInterval` is set appropriately (typically 5-15 minutes) to avoid overloading the DNAC API, as Assurance data can be heavy.

---

## 3. Metrics & Data Collection

Once integrated, VNA pulls a hierarchical object model and associated metrics.

### Inventory Objects
| Object Type | Description |
| :--- | :--- |
| **Site/Building/Floor** | Physical hierarchy imported from DNAC Design. |
| **Network Device** | Switches (Edge/Border/Control), Routers, WLCs. |
| **Access Point** | Wireless APs mapped to floors. |
| **Interface** | Physical interfaces on devices. |
| **SSID** | Wireless LANs broadcasted. |

### Key Performance Metrics
| Metric Group | Specific Metrics | Use Case |
| :--- | :--- | :--- |
| **Device Health** | `CPU Utilization`, `Memory Utilization`, `Reachability Status`, `Device Temperature` | Infrastructure monitoring. |
| **Assurance Health** | `Overall Health Score` (1-10), `Client Health Score`, `Network Health Score` | High-level status "at a glance". |
| **Wireless** | `Client Count`, `Channel Utilization`, `Noise Floor`, `Interference`, `Air Quality` | WiFi troubleshooting. |
| **Client Stats** | `SNR` (Signal-to-Noise Ratio), `RSSI`, `Throughput`, `Retries`, `Connection Score` | End-user experience analysis. |
| **Application** | `Application Health Score`, `Packet Loss`, `Latency` (if App/AVC is enabled) | Application performance triage. |

---

## 4. Triage Workflows & Use Cases

This section outlines how to use the captured data for real-world troubleshooting in the DxNetOps Portal.

### Scenario 1: "Wifi Sucks" - Troubleshooting Wireless Slowness
**Problem**: Users in "Building 3" report slow connection speeds.
**Workflow**:
1.  **Navigate to Site View**: In DxNetOps Portal, drill down to `Sites > Building 3`.
2.  **Check Health Scores**: Look at the aggregate "Wireless Health" score from DNAC. If it's red/orange (Score < 7), there's a systemic issue.
3.  **Drill to APs**: Click on the "Access Points" tab. Sort by `Client Count` (high load) or `Interference` (RF issues).
4.  **Isolate AP**: Select the AP with the lowest health score.
5.  **Correlate**: View `Channel Utilization` vs `Client Count`. High utilization with low clients suggests non-WiFi interference.
6.  **Resolution**: Identify if it's RF interference or just density.

### Scenario 2: SD-Access Fabric Connectivity
**Problem**: A user cannot reach a server in the data center. The user is on a Fabric Edge node.
**Workflow**:
1.  **Topology View**: Use the VNA topology to see the user's Edge switch and its connection to the Border node.
2.  **Underlay Check**: Check standard SNMP metrics (via PM/Spectrum) for the physical link between Edge and Border (IS-IS/Underlay link errors).
3.  **Overlay Check**: Check the DNAC `Reachability` and `Fabric Health` score for the Edge device.
4.  **Result**: If Underlay metrics (Interface Errors) are clean, but Fabric Health is low, the issue is likely LISP/VXLAN control plane related.

### Scenario 3: Application Performance Triage
**Problem**: "Zoom is choppy" for users at a branch.
**Workflow**:
1.  **App Context**: In DxNetOps, go to the `Application Health` dashboard populated by DNAC.
2.  **Filter by App**: Select "Zoom".
3.  **Time Correlation**: Compare the `Application Health Score` drop time with `Network Latency` upstream.
4.  **Path Analysis**: If valid, use the integration to cross-launch into the specific DNAC "Client 360" view for a affected VIP user to see if the issue is standard (WiFi) or individual (Device OS).

---

## 5. Automation & API Integration

For advanced users, you can query these metrics directly from the CAPC (Performance Center) OData API for custom reporting.

**Example OData Query for Health Scores:**
`http://<pc-host>:8181/pc/odata/api/deviceHealthScores?$filter=source eq 'CiscoDNA'`

---

*Authored for the Observability Team - 2026*
