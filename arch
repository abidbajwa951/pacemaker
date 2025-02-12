# SUSE Clustering

## Cluster Components:
- **Pacemaker**: Works as the Resource Manager.
- **Corosync**: Used for internal communication between cluster nodes.

## Cluster Types:
- **Local Cluster**: Cluster within the same data center.
- **Metro Cluster**: Cluster spanning different data centers.
- **GEO Cluster**: Cluster distributed across different geographical locations.

## GEO Cluster:
GEO Cluster based on SLE HA can be considered as an 'overlay' cluster, as the underlying local cluster is managed by the parent cluster. The overlay cluster is managed by the **booth cluster ticket manager**. Each participating site in a GEO Cluster runs a service called **boothd**, which connects to the booth daemon running on other sites and exchanges connectivity details.

An **arbitrator** or a third cluster site helps reach consensus about decisions such as resource failover during a network breakdown. **Boothd** uses **UDP** for faster communication.

## High Availability (HA) Base Components:
- **Corosync**
- **Pacemaker**
- **CRM Shell**
- **Hawk** (Graphical User Interface for cluster management)

## Storage Components:
- **Clustered NFS**
- **Clustered LVM**
- **OCFS2** (Oracle Cluster File System v2)
- **Cluster Managed MDRAID**
- **DRBD** (Distributed Replicated Block Device)
- **Samba** (Clustered file sharing for Windows clients)
- **Ceph** (Highly scalable distributed storage system)

## Load Balancing:
- **Linux IP Virtual Server (LVS)**
- **HA Proxy**
- **Conntrackd** (Connection tracking for load balancing)
- **Rear (Relax and Recover)**: Disaster recovery and backup tool.

## SUSE Linux HA Extension Benefits:
- Maintain **business continuity**
- Protect **data integrity**
- Reduce **unplanned downtime**
- Achieve **high availability** for data, applications, and services

## Cluster Terminologies:
1. **High Availability Cluster**: Designed to secure the highest possible availability of services.
2. **Node**: Any physical or virtual machine that is a member of the cluster and operates transparently.
3. **Consensus Cluster Membership (CCM)**: Determines which nodes make up the cluster and shares this information across all nodes.
4. **Quorum**: A cluster partition is considered to have quorum if it has the majority of nodes (votes).
5. **Epoch**: The version number of the cluster configuration/status.
6. **Split Brain**: A scenario where cluster nodes are divided into two or more groups unaware of each other.
7. **Fencing**: Preventing an isolated or failing cluster member from accessing shared resources.
8. **Active/Passive**: Services run on an active node, and passive nodes take over only if the active node fails.
9. **Active/Active**: Services are distributed across all nodes, with each node able to take over the workload of another.

## HA Cluster Architecture:
### 1. Messaging & Membership Layer (Corosync):
- Uses the **Corosync Cluster Engine** for communication.
- Supports **Unicast** (more secure) and **Multicast** (easier to add nodes).
- Runs as a separate **systemd** service.

### 2. Resource Allocation Layer (Pacemaker):
#### - **Cluster Resource Manager (CRM):**
  - Controls resource allocation and cluster communication.
  - Maintains and manages the **Cluster Information Base (CIB)**.
  - One CRM node is elected as the **Designated Coordinator (DC)**.
  - CRM daemon: `crmd`.

#### - **Cluster Resource Types:**
  - **Primitive Resources:** Basic resources managed by the cluster.
  - **Grouped Resources:** Multiple resources managed together.
  - **Clone Resources:** Resources that can run on multiple nodes simultaneously.
  - **Multi-state (Master-Slave) Resources:** A resource that can operate in different roles.

#### - **Failover and Recovery Mechanisms:**
  - Determines how resources are moved in case of failure.
  - Recovery strategies include restart, relocation, and node fencing.

#### - **Cluster Information Base (CIB):**
  - XML representation of cluster configuration and state.
  - Contains definitions for nodes, resources, constraints, and relationships.
  - The **Designated Coordinator (DC)** maintains the master copy, while all nodes hold replicas.

#### - **Designated Coordinator (DC):**
  - Elected among all nodes after a membership change.
  - Decides cluster-wide actions like resource movement or fencing.

#### - **Policy Engine (PE):**
  - Runs on the DC node.
  - Calculates the next state of the cluster based on its current state and configuration.
  - Generates a **transition graph** with resource actions and dependencies.

#### - **Local Resource Manager (LRM):**
  - Receives requests from CRM and interacts with Resource Agents (RA).
  - Tasks performed: **Start, Stop, Monitor**.

### 3. Resource Layer:
- **Resource Agents (RA):** Scripts or programs managing specific services.
  - Examples:
    - **Ipaddr2** (Manages virtual IP addresses)
    - **STONITH** (Fencing mechanism)
- **Monitoring & Logging:**
  - Tools like `crm_mon`, `hawk2`, and `pcs status` help in monitoring.
  - Logs stored in `/var/log/pacemaker.log` and Corosync logs.
- **Security Aspects:**
  - Role-Based Access Control (RBAC) for user restrictions.
  - Authentication mechanisms like SSL and certificates.

## STONITH (Shoot The Other Node In The Head):
- Pacemaker-provided **fencing mechanism**.
- Ensures a failed node is **isolated** before recovery operations start.
- Used to prevent data corruption by denying access to shared storage.

## Distributed Lock Manager (DLM):
- Provides **cluster-wide locking**.
- Allows **Active/Active storage** scenarios.
- Runs as a **resource** on each cluster node, utilizing Pacemaker membership services.

## Cluster Logical Volume Manager (cLVM):
- **Daemon:** `lvmlockd` (replaces `clvmd` in SLE 15 HA).
- Used with **LVM2** in a clustered environment.
- Protects **LVM metadata** on shared storage.

## Network Redundancy and Considerations:
- **Redundant network interfaces** (bonding) prevent communication failures.
- **Dual-ring Corosync configurations** provide additional reliability.

## Advanced Fencing Strategies:
- **IPMI, iLO, and Redfish** for hardware-based fencing.
- Combination fencing for increased reliability.

## Performance Considerations:
- Cluster scalability limits and resource distribution.
- Optimizing cluster communication for minimal latency.

---
