

# 🚀 talos-homelab-core

### *Minimalist, Immutable Kubernetes on Bare Metal*

[![Talos](https://img.shields.io/badge/OS-Talos_Linux-blue?logo=kubernetes&logoColor=white)](https://www.talos.dev/)
[![Kubernetes](https://img.shields.io/badge/K8s-v1.30+-blue?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Status](https://img.shields.io/badge/Status-Active-success)](#)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](#)

**Buzzwords:** `Infrastructure-as-Code` • `GitOps-Ready` • `Immutable OS` • `Security-Hardened` • `Zero-SSH` • `Bare-Metal` • `Sidero Labs`

---

## 📖 Overview
This repository contains a streamlined workflow for deploying a **Talos Linux** cluster. Unlike traditional distributions, Talos is **immutable**, **ephemeral**, and **managed entirely via API**.

### 🏗️ The Architecture


```text
+--------------------------------------+
|        Kubernetes Workloads          |
|  (Pods, Deployments, Services, Helm) |
+--------------------------------------+
                  ▲
                  |
+--------------------------------------+
|          Talos Immutable OS          |
| - Read-only root filesystem          |
| - No SSH / No package installs       |
| - Configured via talosctl only       |
| - Consistent & secure across reboots |
+--------------------------------------+
                  ▲
                  |
+--------------------------------------+
|      Physical / Virtual Machine      |
| - CPU, RAM, Storage, Network         |
+--------------------------------------+
```

### 🛡️ Understanding Immutability <br>
“Immutable” literally means unchangeable. In the context of Talos Linux: <br>
No Direct SSH: You cannot SSH into the node to apt install or manually tweak files. <br>
Read-Only Root FS: The OS is locked at runtime.  <br>
Every reboot restores the OS to a known, perfect state. <br>
API-Driven: All changes (networking, users, K8s settings) are made via YAML configuration and talosctl. <br>
The Analogy: Think of Talos as a Fridge. You can put food inside (Pods/Apps), but you cannot change the internal wiring or the compressor firmware. <br>

### 🛠️ Prerequisites & Setup <br>
### 1️⃣ Install Management Tools (Ubuntu) <br>
On your management machine, install the required binaries: <br>

```
curl -sL [https://talos.dev/install](https://talos.dev/install) | sh
sudo mv talosctl /usr/local/bin/
talosctl version
```
### 👉 Build & Download ISO -> https://factory.talos.dev/ <br>

### 2️⃣ Generate Custom ISO <br>
Use the Talos Factory to include specific drivers for Intel hardware (i915 Graphics & Microcode): <br>
Platform: metal | Arch: amd64 <br>
Extensions: siderolabs/i915, siderolabs/intel-ucode <br>

<img width="919" height="814" alt="Screenshot from 2026-02-19 13-25-25" src="https://github.com/user-attachments/assets/9d99917c-481b-4818-b908-3a541aec6b84" />

Boot your target machine from the ISO. Note the IP address shown on the console. Identify your installation disk: <br>

```
# Replace <NODE_IP> with your actual node IP
talosctl -n <NODE_IP> get disks --insecure -o yaml
```
Generate Cluster Configuration
```
talosctl gen config martinos https://<NODE_IP>:6443
```

<img width="1276" height="948" alt="Screenshot from 2026-02-19 13-12-28" src="https://github.com/user-attachments/assets/ef87310f-4607-478b-8ee1-2a97c4a7219b" /> <br>

Edit controlplane.yaml. <br>

<img width="910" height="409" alt="Screenshot from 2026-02-19 13-16-58" src="https://github.com/user-attachments/assets/377eb63d-82a8-442a-ba91-04b7aebbd59e" /> <br>

Set install.disk to your identified disk (e.g., /dev/nvme0n1). <br>

<img width="910" height="166" alt="Screenshot from 2026-02-19 13-17-35" src="https://github.com/user-attachments/assets/4a4a1475-2b98-4b81-b4a1-0fbd3c933495" />

For single-node clusters, ensure you allow scheduling on the control plane. <br>

###  Apply & Bootstrap
Apply configuration (Installs OS to disk and reboots)
```
talosctl apply-config -n <NODE_IP> --insecure --file controlplane.yaml
```
Wait 2 minutes for reboot, then Bootstrap Kubernetes
```
talosctl bootstrap --nodes <NODE_IP> --endpoints <NODE_IP> --talosconfig=talosconfig
```
Merge Kubeconfig for kubectl access
```
export TALOSCONFIG=talosconfig
talosctl config endpoint <NODE_IP>
talosctl kubeconfig -n <NODE_IP>
```
📉 Essential Management
```
Check Cluster Health: talosctl health -n <NODE_IP>
Monitor Node: talosctl dashboard -n <NODE_IP>
Verify K8s: kubectl get nodes -o wide
Shutdown Node: talosctl shutdown -n <NODE_IP>
```
<img width="919" height="756" alt="Screenshot from 2026-02-19 13-23-48" src="https://github.com/user-attachments/assets/ff44a143-dd64-4d1d-8ced-53c93f0eb007" />

