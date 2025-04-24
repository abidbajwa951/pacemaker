
# Pacemaker Cluster Configuration and Apache Resource Setup

This guide details the step-by-step process to configure a basic Pacemaker cluster on a RHEL 9 environment, including floating IP and Apache web server setup. This also includes testing failover and resource functionality.

---

## 🌐 Environment Overview

| Hostname             | IP Address      | FQDN                    |
|----------------------|------------------|--------------------------|
| Workstation          | 192.168.159.128 | workstation.example.com |
| Node A               | 192.168.159.129 | nodea.example.com       |
| Node B               | 192.168.159.130 | nodeb.example.com       |
| Node C               | 192.168.159.131 | nodec.example.com       |
| Floating IP Address  | 192.168.159.200 | -                        |

---

## ✅ Prerequisites

1. Two or more RHEL 9 nodes that can communicate over the same network.
2. A floating IP (`192.168.159.200`) on the same network as the nodes.
3. Hostnames of the nodes are added in `/etc/hosts` file.

---

## 🔧 Pacemaker Cluster Initial Configuration

### 1. Enable High Availability Repositories
```bash
subscription-manager repos --enable=rhel-9-for-x86_64-highavailability-rpms
yum update
```

### 2. Install Required Packages
```bash
dnf install pcs pacemaker fence-agents-all -y
```

### 3. Start and Enable the pcsd Service
```bash
systemctl start pcsd.service
systemctl enable pcsd.service
```

### 4. Configure Firewall for High Availability
```bash
firewall-cmd --permanent --add-service=high-availability
firewall-cmd --reload
```

### 5. Set hacluster User Password
```bash
echo redhat | passwd --stdin hacluster
```

---

## 🌐 Apache Configuration

### 1. Install Apache and Wget
```bash
dnf install -y httpd wget
```

### 2. Allow HTTP through Firewall
```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

### 3. Create a Simple Web Page
```bash
cat <<-END >/var/www/html/index.html
<html>
<body>My Test Site - $(hostname)</body>
</html>
END
```

### 4. Configure Apache Server Status for Monitoring
```bash
cat <<-END > /etc/httpd/conf.d/status.conf
<Location /server-status>
SetHandler server-status
Order deny,allow
Deny from all
Allow from 127.0.0.1
Allow from ::1
</Location>
END
```

---

## ⚙️ Pacemaker Cluster Configuration

### 1. Authenticate Cluster Nodes
```bash
pcs host auth nodea.example.com nodeb.example.com nodec.example.com
```

### 2. Create and Start Cluster
```bash
pcs cluster setup my_cluster --start nodea.example.com nodeb.example.com nodec.example.com
```

### 3. Disable Fencing (Optional for Testing)
```bash
pcs property set stonith-enabled=false
```

### 4. Create Cluster Resources (Floating IP and Apache)
```bash
pcs resource create ClusterIP ocf:heartbeat:IPaddr2 ip=192.168.159.200 --group apachegroup
pcs resource create WebSite ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl="http://localhost/server-status" --group apachegroup
```

---

## 🧪 Testing Cluster Functionality

### 1. Simulate Apache Failure and Test Failover
```bash
pcs status
killall -9 httpd
curl http://192.168.159.200
pcs status
```

### 2. Place a Node into Standby Mode and Test Failover
```bash
pcs status
pcs node standby nodea.example.com
pcs status
curl http://192.168.159.200
```

---

