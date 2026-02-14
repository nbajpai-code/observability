# Broadcom DxNetOps 25.4.4 Installation Guide (CAAS)
**Component**: Container as a Service (CaaS) / Kubernetes

This document details the deployment of the CaaS layer required for modern NetOps microservices (required for Spectrum integration, NFA, and advanced analytics).

## 1. Prerequisites
*   **Platform**: Kubernetes 1.25+ or Red Hat OpenShift 4.10+.
*   **Resources (Minimum)**:
    *   3 Worker Nodes (16 vCPU, 64GB RAM each).
    *   1 Master Node (if managing own K8s).
*   **Storage**: Persistent Volume (PV) support (NFS/StorageClass).

## 2. Prepare the Deployment Host
This host runs the Ansible playbooks to deploy CaaS to the target cluster.
1.  **Download Media**: `dx-caas-installer-25.4.4.tar.gz`
2.  **Extract**:
    ```bash
    mkdir dx-caas
    tar -xvf dx-caas-installer-25.4.4.tar.gz -C dx-caas
    cd dx-caas
    ```
3.  **Load Docker Images** (if air-gapped):
    ```bash
    ./offline-image-load.sh
    ```

---

## 3. Configuration (vars.yml)
Edit the `vars.yml` file to define your environment.

```yaml
# vars.yml example
global:
  environment: "production"
  domain: "netops.local"

kubernetes:
  namespace: "dxnetops"
  storage_class: "nfs-client"  # Ensure this matches your K8s StorageClass

ingress:
  enabled: true
  hostname: "caas.netops.local"
  tls:
    enabled: true
    cert_path: "/path/to/cert.crt"
    key_path: "/path/to/key.key"

components:
  doi:  # Digital Operational Intelligence
    enabled: false
  acc:  # App Context
    enabled: true
```

---

## 4. Deploy CaaS
Broadcom provides an `install.sh` wrapper or direct Ansible execution.

### Standard Deployment
```bash
./install.sh --config vars.yml
```

### Verification
Check the status of the pods in the namespace:
```bash
kubectl get pods -n dxnetops
```
*Expected Output*: All pods (e.g., `pulsar`, `druid`, `gateway`) should be `Running` or `Completed`.

---

## 5. Post-Install: Register with Portal
1.  Obtain the **CaaS Ingress URL** (e.g., `https://caas.netops.local`).
2.  Log in to **NetOps Portal**.
3.  Navigate to **Administration > Data Sources > Text/CAAS**.
4.  Add the CaaS Source using the URL and the secret token generated during install.

*Document generated for `nbajpai-code/observability`*
