
# NFS with LVM in Pacemaker Cluster (RHEL 9)

## Environment

| Role         | IP Address       | Hostname              |
|--------------|------------------|------------------------|
| Workstation  | 192.168.159.128  | workstation.example.com |
| Node A       | 192.168.159.129  | nodea.example.com       |
| Node B       | 192.168.159.130  | nodeb.example.com       |
| Node C       | 192.168.159.131  | nodec.example.com       |
| Floating IP  | 192.168.159.200  | -                       |

**Disk:** Additional 5GB disk on nodea.

## Prerequisites

1. Three RHEL 9 nodes that can communicate with each other.
2. A floating IP on the same network as the nodes.
3. Hostnames are configured in `/etc/hosts`.
4. A shared or attached additional disk between the nodes.

## Pacemaker Cluster Initial Configuration

### 1. Enable High Availability Repositories

```bash
subscription-manager repos --enable=rhel-9-for-x86_64-highavailability-rpms
yum update -y
```

### 2. Install Required Packages

```bash
dnf install pcs pacemaker fence-agents-all -y
```

### 3. Start and Enable Services

```bash
systemctl start pcsd.service
systemctl enable pcsd.service
systemctl enable pacemaker.service
systemctl enable corosync.service
```

### 4. Configure Firewall

```bash
firewall-cmd --permanent --add-service=high-availability
firewall-cmd --reload
```

### 5. Set hacluster Password

```bash
echo redhat | passwd --stdin hacluster
```

## VMware VMX File Changes

### Disable Locking

```
disk.locking = "FALSE"
diskLib.dataCacheMaxSize = "0"
diskLib.dataCacheMaxReadAheadSize = "0"
diskLib.dataCacheMinReadAheadSize = "0"
diskLib.dataCachePageSize = "4096"
disk.EnableUUID = "TRUE"
```

### Shared Disk Configuration

```
scsi1.sharedBus = "virtual"
scsi1:0.present = "TRUE"
scsi1:0.fileName = "shared.vmdk"
scsi1:0.mode = "independent-persistent"
scsi1:0.deviceType = "disk"
```

## LVM Setup

### Configure LVM on All Nodes

```bash
cp /etc/lvm/lvm.conf /etc/lvm/lvm.conf-bck
vi /etc/lvm/lvm.conf
# Set the following:
system_id_source = "uname"
use_devicesfile = 1
```

### Create LVM

```bash
parted /dev/sda
pvcreate /dev/sda1
vgcreate --setautoactivation n my_vg /dev/sda1
lvcreate -l 100%FREE -n my_lv my_vg
mkfs.xfs /dev/my_vg/my_lv
```
#### Note: Reboot all nodes after creating LVM.

### Add Device to All Nodes

```bash
lvmdevices --adddev /dev/sda1
```

## NFS Configuration

**Required Packages (on all nodes):**  
Run this on all cluster nodes:
```bash
dnf install -y nfs-utils
```

**Enable and Start Required Services:**
```bash
systemctl enable --now nfs-server
systemctl enable --now rpcbind
```

```bash
mkdir /nfsshare
lvchange -ay my_vg/my_lv
mount /dev/my_vg/my_lv /nfsshare
mkdir -p /nfsshare/exports/export1
mkdir -p /nfsshare/exports/export2
touch /nfsshare/exports/export1/clientdatafile1
touch /nfsshare/exports/export2/clientdatafile2
```

### Configure NFS Firewall

```bash
firewall-cmd --permanent --add-service=nfs,rpcbind,mountd
firewall-cmd --reload
```

## Pacemaker Configuration

### 1. Authenticate Nodes

```bash
pcs host auth nodea.example.com nodeb.example.com nodec.example.com
```

### 2. Create and Start Cluster

```bash
pcs cluster setup my_cluster --start nodea.example.com nodeb.example.com nodec.example.com
```

### 3. Disable Fencing (Optional)

```bash
pcs property set stonith-enabled=false
```

### 4. Create Resources

```bash
pcs resource create my_lvm ocf:heartbeat:LVM-activate vgname=my_vg vg_access_mode=system_id --group nfsgroup
pcs resource create nfsshare Filesystem device=/dev/my_vg/my_lv directory=/nfsshare fstype=xfs --group nfsgroup
pcs resource create nfs-daemon nfsserver nfs_shared_infodir=/nfsshare/nfsinfo nfs_no_notify=true --group nfsgroup
pcs resource create nfs-root exportfs clientspec=192.168.122.0/255.255.255.0 options=rw,sync,no_root_squash directory=/nfsshare/exports fsid=0 --group nfsgroup
pcs resource create nfs-export1 exportfs clientspec=192.168.122.0/255.255.255.0 options=rw,sync,no_root_squash directory=/nfsshare/exports/export1 fsid=1 --group nfsgroup
pcs resource create nfs-export2 exportfs clientspec=192.168.122.0/255.255.255.0 options=rw,sync,no_root_squash directory=/nfsshare/exports/export2 fsid=2 --group nfsgroup
pcs resource create nfs_ip IPaddr2 ip=192.168.122.200 cidr_netmask=24 --group nfsgroup
pcs resource create nfs-notify nfsnotify source_host=192.168.122.200 --group nfsgroup
```

