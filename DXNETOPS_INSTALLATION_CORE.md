# Broadcom DxNetOps 25.4.4 Installation Guide (Core Components)
**Target Architecture**: VMware vSphere HA/FT (No App-Level HA)

This document outlines the installation steps for the core DxNetOps components on Linux (RHEL 8.x/9.x). As per the `DXNETOPS_VMWARE_HA` strategy, components are installed in "Standalone" mode, relying on vSphere for high availability.

## 1. Pre-Installation Requirements
Ensure all VMs meet the following baseline specifications (Medium Deployment):
*   **OS**: Red Hat Enterprise Linux 8.8 or 9.2+
*   **User**: Root or Sudo user required for installation.
*   **Network**: Static IP, Hostname resolution (DNS) configured.
*   **Firewall**: Ports 80, 443, 8181, 8581, 9990 allowed.

---

## 2. NetOps Portal (Performance Center)
The web-based user interface.

### Steps
1.  **Download Media**: `CAPerfCenterSetup.bin`
2.  **Prepare Environment**:
    ```bash
    # Install dependencies
    yum install -y fontconfig liberation-fonts
    # Set file limits
    ulimit -n 65536
    ```
3.  **Run Installer**:
    ```bash
    chmod +x CAPerfCenterSetup.bin
    ./CAPerfCenterSetup.bin -i console
    ```
4.  **Follow Prompts**:
    *   **Install Directory**: `/opt/CA/PerformanceCenter`
    *   **HTTPS**: Enable (recommended). Generate self-signed or import cert.
    *   **Services**: Start services automatically.
5.  **Validation**: Access `https://<portal_hostname>:8181/pc`

---

## 3. Data Aggregator (DA)
The polling engine. *Note: Protected by vSphere Fault Tolerance (FT) if configured.*

### Steps
1.  **Download Media**: `DataAggregatorSetup.bin`
2.  **Run Installer**:
    ```bash
    chmod +x DataAggregatorSetup.bin
    ./DataAggregatorSetup.bin -i console
    ```
3.  **Follow Prompts**:
    *   **Deployment Type**: Select **Standalone** (Do NOT select Cluster/HA).
    *   **Data Repository**: Enter the Hostname/IP of the Vertica Database node.
    *   **ActiveMQ**: Configure internal broker.
4.  **Post-Install**:
    *   The DA will automatically sync with the Data Repository.

---

## 4. Data Collector (DC)
Collects SNMP/API data. *Note: Protected by vSphere HA.*

### Steps
1.  **Download Media**: `DataCollectorSetup.bin`
2.  **Run Installer**:
    ```bash
    chmod +x DataCollectorSetup.bin
    ./DataCollectorSetup.bin -i console
    ```
3.  **Follow Prompts**:
    *   **Data Aggregator Host**: Enter the IP/Hostname of the DA installed above.
    *   **Tenant**: Default Tenant (unless multi-tenancy is required).
4.  **Sizing**: Allocate Java Heap memory based on the "Medium" profile (e.g., 4GB-8GB).

---

## 5. Proxy Server (Nginx)
Used for reliable UI access and load balancing if scaling out PCs (though usually 1 PC is sufficient).

### Steps
1.  **Install Nginx**:
    ```bash
    yum install -y nginx
    ```
2.  **Configure upstream**:
    Edit `/etc/nginx/nginx.conf`:
    ```nginx
    upstream netops_portal {
        server <portal_ip>:8181;
    }
    server {
        listen 443 ssl;
        location / {
            proxy_pass https://netops_portal;
        }
    }
    ```
3.  **Start Service**:
    ```bash
    systemctl enable --now nginx
    ```

---

## 6. Post-Installation Integration
1.  Log in to **NetOps Portal** (`admin`/`admin` default).
2.  Navigate to **Administration > Data Sources > Data Aggregator**.
3.  The DA should auto-discover if the installer pointed correctly. If not, click **Add** and provide the DA Hostname.

*Document generated for `nbajpai-code/observability`*
