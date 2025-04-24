
# Pacemaker Cluster Management - Command Summary

This document provides a categorized summary of commonly used `pcs` commands for managing a Pacemaker cluster. These include configuration, updates, resource management, and status checks.

---

## 🗂️ CIB (Cluster Information Base) Management

### Check Current CIB Configuration
```bash
pcs cluster cib
```
Displays the current CIB XML content.

### Save CIB to a File
```bash
pcs cluster cib original.xml
```
Exports the current CIB configuration to a file named `original.xml`.

### Make a Copy of CIB File
```bash
cp original.xml updated.xml
```
Creates a working copy of the CIB for modifications.

---

## ⚙️ Modifying the CIB File

### Create/Update a Resource in CIB
```bash
pcs -f updated.xml resource create VirtualIP ocf:heartbeat:IPaddr2 ip=192.168.0.120 op monitor interval=30s
```
Adds a `VirtualIP` resource with monitoring parameters to the `updated.xml` file.

---

## 📤 Pushing CIB Changes

### Push Only Changes (Diff-Based)
```bash
pcs cluster cib-push updated.xml diff-against=original.xml
```
Applies only the changes between the original and updated CIB files.

### Push Entire Updated CIB
```bash
pcs cluster cib-push updated.xml
```
Overwrites the current CIB with the contents of `updated.xml`.

---

## 🔍 Monitoring and Status

### Wait Until Cluster Status is Updated
```bash
pcs status wait
```
Waits until all changes are applied and the cluster reaches a stable state.

### Display Full Cluster Configuration
```bash
pcs config
```
Shows the full configuration of the cluster.

### Query Specific Resource Status
```bash
pcs status query resource ClusterIP is-state started
```
Checks if the resource named `ClusterIP` is currently in the `started` state.

---

## 🛠️ Corosync Configuration

### Update `corosync.conf` File
```bash
pcs cluster config update [transport options] [compression options] [crypto options] [totem options] [--corosync_conf path]
```
Updates the `corosync.conf` file with specified options.

### Display Corosync Configuration
```bash
pcs cluster corosync
```
Displays the current Corosync configuration.

### Show Cluster Config Again (includes Corosync and Pacemaker settings)
```bash
pcs cluster config
```

---

