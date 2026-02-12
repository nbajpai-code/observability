# Broadcom DxNetOps High Availability & Disaster Recovery

This document outlines a High Availability (HA) and Disaster Recovery (DR) architecture for a multi-site Broadcom DxNetOps deployment, with a specific focus on optimizing Vertica database "hot cutover" to minimize mean time to recovery (MTTR).

## Multi-Site Architecture Overview

A robust deployment typically spans two geographically separated data centers:
1.  **Primary Site (Active):** Handles all production traffic and monitoring loads.
2.  **Secondary Site (Hot Standby/DR):** A mirrored environment ready to take over operations immediately.

### Key Components
- **Data Aggregator (DA):** The polling engine.
- **Data Collector (DC):** Distributed collectors close to managed devices.
- **Performance Center (PC):** The unified web UI.
- **Data Repository (Vertica):** The columnar database storing massive scale performance metrics.

---

## Vertica Database High Availability (Cluster Level)

Before addressing site failover, the primary Vertica cluster itself must be resilient.

1.  **K-Safety Configuration:** Ensure K-Safety is set to **K=1**.
    - This requires a minimum of **3 nodes** in the cluster.
    - Data is replicated across nodes, allowing the cluster to survive the loss of a single node without downtime.
2.  **Fault Groups:** Define fault groups to ensure replica data is stored on different physical racks or reliable zones within the same data center.

---

## Vertica "Hot Cutover" Strategy (Site Level)

To achieve a "hot cutover" with minimal downtime, the traditional backup/restore method is too slow. The recommended strategy is **Dual Ingestion (Active-Active Data Load)**.

### Strategy: Dual Ingestion

Instead of replicating the database files (which can be laggy and heavy), this approach involves sending the data stream to **both** the Primary and Secondary sites simultaneously.

#### Architecture
1.  **Dual Data Collectors:** Configure Data Collectors to feed data to both the Primary DA/Vertica and Secondary DA/Vertica clusters concurrently.
2.  **Independent Clusters:** The Primary Site and Secondary Site run completely independent Vertica clusters.
3.  **Synchronization:** Both clusters hold the same data in near real-time because they ingest the same streams.

#### Benefits
- **Zero RPO (Recovery Point Objective):** Since both sites ingest data simultaneously, the secondary site is always up-to-date.
- **Instant RTO (Recovery Time Objective):** There is no need to "restore" or "replay" logs. The secondary database is already live and operational.
- **Hot Cutover:** Failover is strictly a standard DNS or Load Balancer switch for the User Interface (Performance Center).

### Alternative: CopyCluster (Active-Passive)

If Dual Ingestion is resource-prohibitive, use the `vbr` tool's `copycluster` capability.

1.  **Replication:** Schedule aggressive differential replications using `vbr --task copycluster` from Primary to Secondary.
2.  **RPO:** Limited by the frequency of replication (e.g., every 15 minutes).
3.  **Failover:**
    - Stop replication.
    - Promote the Secondary cluster.
    - Repoint Data Aggregators to the Secondary cluster (this takes time).

**Recommendation:** For critical "hot cutover" requirements, **Dual Ingestion** is the only method that guarantees near-zero failover time.

---

## Operational Workflow for Failover

### Scenario: Primary Site Failure

1.  **Detection:** Global Traffic Manager (GTM) or Load Balancer detects Primary Site PC/DA is unresponsive.
2.  **Traffic Redirection:**
    - **User Traffic:** GTM redirects user web traffic (HTTPS) to the Secondary Site Performance Center.
    - **Device Traffic:** If utilizing dual collectors, they continue sending to the Secondary Site without interruption.
    - **Client Connections:** Configure JDBC connection strings in the application server to support failover lists:
      `jdbc:vertica://Node1,Node2,Node3/DBName?backup_host_list=SecNode1,SecNode2,SecNode3`
3.  **Cutover:**
    - Since the Secondary Vertica is already hydrated (Dual Ingestion), dashboards populate immediately.
    - No database startup or recovery phases are required.

## Summary of Optimization Techniques

| Feature | Design Decision | Benefit |
| :--- | :--- | :--- |
| **Ingestion** | Dual Ingestion (Active-Active) | Eliminates database restore time; ensures 0 data loss. |
| **Cluster HA** | 3+ Nodes, K=1 Safety | Survives single node failure within a site. |
| **Client Conn** | JDBC Backup Host List | Automates client-side connection failover. |
| **DNS** | Short TTL, Health Checks | Rapid redirection of user traffic to the standby UI. |
