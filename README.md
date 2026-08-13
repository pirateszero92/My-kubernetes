# 🚀 Talos OS Upgrade Guide (Kubernetes Cluster)

เอกสารคู่มือการทำ **Rolling Upgrade Talos OS** บน Kubernetes Cluster แบบ **Zero Downtime** โดยเรียงลำดับการทำความสะอาดและอัปเกรด Control Plane และ Worker Nodes ทีละ Node อย่างปลอดภัย

---

## 📌 สรุปข้อมูล Cluster

* **Kubernetes Version:** `v1.35.2`
* **Talos OS Target Version:** `v1.13.8`
* **Installer Image:** `ghcr.io/siderolabs/installer:v1.13.8`

### 🖥️ รายชื่อ Nodes ทั้งหมดใน Cluster

| Node Name | Role | IP Address | Target Version |
| :--- | :--- | :--- | :--- |
| **talos-ok0-rtp** | Control Plane (Node 1) | `10.1.1.111` | `v1.13.8` *(Completed)* |
| **talos-8y5-pkx** | Control Plane (Node 2) | `10.1.1.112` | `v1.13.8` |
| **talos-uin-5l8** | Control Plane (Node 3) | `10.1.1.113` | `v1.13.8` |
| **talos-tzn-az7** | Worker (Node 1) | `10.1.1.114` | `v1.13.8` |
| **talos-uwd-kso** | Worker (Node 2) | `10.1.1.115` | `v1.13.8` |
| **talos-vwj-eqr** | Worker (Node 3) | `10.1.1.116` | `v1.13.8` |
| **talos-3vj-wzf** | Worker (Node 4) | `10.1.1.117` | `v1.13.8` |

---

## 📋 1. การเตรียมความพร้อม (Prerequisites)

1. **ติดตั้ง `talosctl`** (กรณีรันบนเครื่องใหม่/WSL):
   ```bash
   curl -sL https://talos.dev/support/install | sh
   ```
2. **ตั้งค่า `TALOSCONFIG`** ให้ชี้ไปยังไฟล์ `talosconfig` ที่ถูกต้อง:
   ```bash
   export TALOSCONFIG=/path/to/your/talosconfig
   ```
3. **ทดสอบการเชื่อมต่อกับ Node:**
   ```bash
   talosctl version --nodes 10.1.1.111 --endpoints 10.1.1.111
   ```

---

## 🔄 2. ขั้นตอนการอัปเกรด (Rolling Upgrade Step-by-Step)

⚠️ **กฎสำคัญ:** 
* ต้องทำ **ทีละ 1 Node** เท่านั้น
* ทำ **Control Plane ให้ครบทั้ง 3 เครื่องก่อน** แล้วจึงเริ่มทำ **Worker Nodes**
* ตรวจสอบให้ etcd และ Cluster Health เป็น `OK` ก่อนข้ามไปทำ Node ถัดไปเสมอ

---

### 🟢 Phase 1: Control Plane Nodes

#### 1️⃣ Node 1: `talos-ok0-rtp` (IP: `10.1.1.111`) *(ทำเสร็จเรียบร้อยแล้ว)*
```bash
# 1. Cordon และ Drain
kubectl cordon talos-ok0-rtp
kubectl drain talos-ok0-rtp --ignore-daemonsets --delete-emptydir-data --force

# 2. Upgrade Talos OS
talosctl upgrade --nodes 10.1.1.111 --endpoints 10.1.1.111 --image ghcr.io/siderolabs/installer:v1.13.8 --wait

# 3. Check Health
talosctl health --nodes 10.1.1.111 --endpoints 10.1.1.111

# 4. Uncordon & Verify Pods
kubectl uncordon talos-ok0-rtp
kubectl get pods -A --field-selector spec.nodeName=talos-ok0-rtp -o wide
```

---

#### 2️⃣ Node 2: `talos-8y5-pkx` (IP: `10.1.1.112`)
```bash
# 1. Cordon และ Drain
kubectl cordon talos-8y5-pkx
kubectl drain talos-8y5-pkx --ignore-daemonsets --delete-emptydir-data --force

# 2. Upgrade Talos OS
talosctl upgrade --nodes 10.1.1.112 --endpoints 10.1.1.112 --image ghcr.io/siderolabs/installer:v1.13.8 --wait

# 3. Check Health
talosctl health --nodes 10.1.1.112 --endpoints 10.1.1.112

# 4. Uncordon & Verify Pods
kubectl uncordon talos-8y5-pkx
kubectl get pods -A --field-selector spec.nodeName=talos-8y5-pkx -o wide
```

---

#### 3️⃣ Node 3: `talos-uin-5l8` (IP: `10.1.1.113`)
```bash
# 1. Cordon และ Drain
kubectl cordon talos-uin-5l8
kubectl drain talos-uin-5l8 --ignore-daemonsets --delete-emptydir-data --force

# 2. Upgrade Talos OS
talosctl upgrade --nodes 10.1.1.113 --endpoints 10.1.1.113 --image ghcr.io/siderolabs/installer:v1.13.8 --wait

# 3. Check Health
talosctl health --nodes 10.1.1.113 --endpoints 10.1.1.113

# 4. Uncordon & Verify Pods
kubectl uncordon talos-uin-5l8
kubectl get pods -A --field-selector spec.nodeName=talos-uin-5l8 -o wide
```

---

### 🟡 Phase 2: Worker Nodes

#### 4️⃣ Worker 1: `talos-tzn-az7` (IP: `10.1.1.114`)
```bash
# 1. Cordon และ Drain
kubectl cordon talos-tzn-az7
kubectl drain talos-tzn-az7 --ignore-daemonsets --delete-emptydir-data --force

# 2. Upgrade Talos OS
talosctl upgrade --nodes 10.1.1.114 --endpoints 10.1.1.114 --image ghcr.io/siderolabs/installer:v1.13.8 --wait

# 3. Uncordon & Verify Pods
kubectl uncordon talos-tzn-az7
kubectl get pods -A --field-selector spec.nodeName=talos-tzn-az7 -o wide
```

---

#### 5️⃣ Worker 2: `talos-uwd-kso` (IP: `10.1.1.115`)
```bash
# 1. Cordon และ Drain
kubectl cordon talos-uwd-kso
kubectl drain talos-uwd-kso --ignore-daemonsets --delete-emptydir-data --force

# 2. Upgrade Talos OS
talosctl upgrade --nodes 10.1.1.115 --endpoints 10.1.1.115 --image ghcr.io/siderolabs/installer:v1.13.8 --wait

# 3. Uncordon & Verify Pods
kubectl uncordon talos-uwd-kso
kubectl get pods -A --field-selector spec.nodeName=talos-uwd-kso -o wide
```

---

#### 6️⃣ Worker 3: `talos-vwj-eqr` (IP: `10.1.1.116`)
```bash
# 1. Cordon และ Drain
kubectl cordon talos-vwj-eqr
kubectl drain talos-vwj-eqr --ignore-daemonsets --delete-emptydir-data --force

# 2. Upgrade Talos OS
talosctl upgrade --nodes 10.1.1.116 --endpoints 10.1.1.116 --image ghcr.io/siderolabs/installer:v1.13.8 --wait

# 3. Uncordon & Verify Pods
kubectl uncordon talos-vwj-eqr
kubectl get pods -A --field-selector spec.nodeName=talos-vwj-eqr -o wide
```

---

#### 7️⃣ Worker 4: `talos-3vj-wzf` (IP: `10.1.1.117`)
```bash
# 1. Cordon และ Drain
kubectl cordon talos-3vj-wzf
kubectl drain talos-3vj-wzf --ignore-daemonsets --delete-emptydir-data --force

# 2. Upgrade Talos OS
talosctl upgrade --nodes 10.1.1.117 --endpoints 10.1.1.117 --image ghcr.io/siderolabs/installer:v1.13.8 --wait

# 3. Uncordon & Verify Pods
kubectl uncordon talos-3vj-wzf
kubectl get pods -A --field-selector spec.nodeName=talos-3vj-wzf -o wide
```

---

## 🔍 3. ตรวจสอบความเรียบร้อยหลังการอัปเกรดครบทุก Node

1. **เช็คสถานะและเวอร์ชันของทุก Node:**
   ```bash
   kubectl get nodes -o wide
   ```
2. **เช็คสุขภาพของ Cluster โดยรวม:**
   ```bash
   talosctl health --nodes 10.1.1.111 --endpoints 10.1.1.111
   ```
3. **เช็คสถานะ Pods ทั้งหมดใน Cluster:**
   ```bash
   kubectl get pods -A
   ```
