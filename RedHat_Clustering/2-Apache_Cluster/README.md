
# Pacemaker Cluster with Apache and Shared LVM Configuration

This guide covers the configuration of a Pacemaker cluster with a shared LVM setup for hosting an Apache web server. It includes instructions for initial setup, disk configuration, Apache setup, and Pacemaker resource management.

---

## 🌐 Environment Overview

| Hostname     | IP Address      | FQDN                    |
|--------------|------------------|--------------------------|
| Workstation  | 192.168.159.128 | workstation.example.com |
| Node A       | 192.168.159.129 | nodea.example.com       |
| Node B       | 192.168.159.130 | nodeb.example.com       |
| Node C       | 192.168.159.131 | nodec.example.com       |
| Floating IP  | 192.168.159.200 | -                        |

### Disk Information
- Node A has an additional 5GB disk intended for shared use.

---

## ✅ Prerequisites

1. Three RHEL 9 nodes with network connectivity.
2. Floating IP configured on the same network.
3. Hostnames configured in `/etc/hosts`.
4. A shared or attachable disk between nodes for LVM setup.

---

## 🔧 Pacemaker Cluster Initial Setup

### 1. Enable High Availability Repos
```bash
subscription-manager repos --enable=rhel-9-for-x86_64-highavailability-rpms
yum update
```

### 2. Install Necessary Packages
```bash
dnf install pcs pacemaker fence-agents-all -y
```

### 3. Enable and Start Cluster Services
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

---

## 💾 VMware VMX File Configuration for Disk Sharing

### Modify the `.vmx` file:
```text
# Disable locking
disk.locking = "FALSE"
diskLib.dataCacheMaxSize = "0"
diskLib.dataCacheMaxReadAheadSize = "0"
diskLib.dataCacheMinReadAheadSize = "0"
diskLib.dataCachePageSize = "4096"
disk.EnableUUID = "TRUE"

# Shared disk properties
scsi1.sharedBus = "virtual"
scsi1:0.present = "TRUE"
scsi1:0.fileName = "shared.vmdk"
scsi1:0.mode = "independent-persistent"
scsi1:0.deviceType = "disk"
```

---

## 📦 LVM Setup for Shared Disk

### 1. Edit LVM Configuration
```bash
cp /etc/lvm/lvm.conf /etc/lvm/lvm.conf-bck
vi /etc/lvm/lvm.conf
# Change/add the following:
system_id_source = "uname"
use_devicesfile = 1
```

### 2. Check LVM System ID
```bash
lvm systemid
uname -n
```

### 3. Create LVM Partition and Volume
```bash
parted /dev/sda
pvcreate /dev/sda1
vgcreate --setautoactivation n my_vg /dev/sda1
vgs -o+systemid
lvcreate -l 100%FREE -n my_lv my_vg
mkfs.xfs /dev/my_vg/my_lv
```

### 4. Add Devices to All Nodes
```bash
lvmdevices --adddev /dev/sda1
```

---

## 🌐 Apache Setup

### 1. Install Apache
```bash
dnf install -y httpd wget
```

### 2. Configure Firewall
```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

### 3. Configure Apache Server Status
```bash
cat <<-END > /etc/httpd/conf.d/status.conf
<Location /server-status>
    SetHandler server-status
    Require local
</Location>
END
```

### 4. Prepare Web Content on LVM
```bash
lvchange -ay my_vg/my_lv
mount /dev/my_vg/my_lv /var/www/
mkdir -p /var/www/html /var/www/cgi-bin /var/www/error
restorecon -R /var/www
cat <<-END >/var/www/html/index.html
<html><body>Hello</body></html>
END
umount /var/www
```

### 5. Modify Apache Logrotate
Edit `/etc/logrotate.d/httpd`:
Replace:
```bash
/bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
```
With:
```bash
/usr/bin/test -f /var/run/httpd-Website.pid >/dev/null 2>/dev/null &&
/usr/bin/ps -q $(/usr/bin/cat /var/run/httpd-Website.pid) >/dev/null 2>/dev/null &&
/usr/sbin/httpd -f /etc/httpd/conf/httpd.conf -c "PidFile /var/run/httpd-Website.pid" -k graceful > /dev/null 2>/dev/null || true
```

---

## ⚙️ Pacemaker Cluster Resource Configuration

### 1. Authenticate Cluster Nodes
```bash
pcs host auth nodea.example.com nodeb.example.com nodec.example.com
```

### 2. Create and Start the Cluster
```bash
pcs cluster setup my_cluster --start nodea.example.com nodeb.example.com nodec.example.com
```

### 3. Disable Fencing (for testing)
```bash
pcs property set stonith-enabled=false
```

### 4. Define Cluster Resources
```bash
pcs resource create my_lvm ocf:heartbeat:LVM-activate vgname=my_vg vg_access_mode=system_id --group apachegroup
pcs resource create my_fs Filesystem device="/dev/my_vg/my_lv" directory="/var/www" fstype="xfs" --group apachegroup
pcs resource create VirtualIP IPaddr2 ip=192.168.159.200 cidr_netmask=24 --group apachegroup
pcs resource create Website apache configfile="/etc/httpd/conf/httpd.conf" statusurl="http://127.0.0.1/server-status" --group apachegroup
```

---

