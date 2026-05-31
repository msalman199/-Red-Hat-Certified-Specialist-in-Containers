# 🛠️ Troubleshooting Common Container Issues with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Podman-Container_Debugging-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Linux-Troubleshooting-blue?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/SELinux-Security-red?style=for-the-badge&logo=redhat" />
  <img src="https://img.shields.io/badge/Networking-Debugging-green?style=for-the-badge&logo=gnome-terminal" />
</p>

---

# 📖 Overview

Container issues are common in real-world DevOps environments. Problems may arise from:

- 🧠 CPU or memory limits  
- 🔐 SELinux restrictions  
- 🌐 Network conflicts  
- 🔑 Permission issues  

This lab teaches how to **diagnose, debug, and fix container runtime problems using Podman tools**.

---

# 🎯 Objectives

By the end of this lab, you will be able to:

✅ Diagnose CPU & memory constraints  
✅ Resolve SELinux permission denials  
✅ Debug network and port conflicts  
✅ Fix container permission issues  
✅ Use Podman inspection tools effectively  

---

# 🛠️ Prerequisites

| Requirement | Description |
|------------|-------------|
| 🐳 Podman | Installed (`dnf install podman`) |
| 🐧 Linux System | RHEL / Fedora / Ubuntu |
| 🔐 Sudo Access | Required for SELinux & system tools |
| 💻 Terminal | Command-line access |
| 📚 Basic Knowledge | Containers & Linux fundamentals |

---

# 📌 Task 1: Diagnose Resource Constraints

---

# 🟢 Subtask 1.1: Inspect Container Resource Limits

## 🔹 List Running Containers

```bash
podman ps
```

📊 Outcome:
Shows container IDs, names, and status.

---

## 🔹 Inspect CPU & Memory Limits

```bash
podman inspect <container_id> | grep -i "memory\|cpu"
```

📊 Outcome:
Displays configured resource limits.

---

## 🔹 Monitor Live Usage

```bash
podman stats <container_id>
```

📊 Outcome:
Live CPU, memory, and network usage.

---

# 🟢 Subtask 1.2: Adjust Resource Limits

## 🔹 Update Memory Limit

```bash
podman update --memory 512m <container_id>
```

---

## ⚠️ Troubleshooting Tip

If container crashes:

- Increase memory limit
- Check application logs
- Review CPU throttling

---

# 📌 Task 2: Resolve SELinux Denials

---

# 🟢 Subtask 2.1: Check SELinux Audit Logs

```bash
sudo ausearch -m avc -ts recent
```

📊 Outcome:
Shows SELinux denial events.

Look for:

```text
container_t
```

---

## 🔹 Temporarily Disable SELinux (Debug Only)

```bash
sudo setenforce 0
```

⚠️ Warning:
Re-enable after testing:

```bash
sudo setenforce 1
```

---

# 🟢 Subtask 2.2: Fix SELinux Labels

```bash
sudo chcon -Rt container_file_t /path/to/volume
```

---

## 🧠 SELinux Concept Flow

```text
File/Volume
    │
    ▼
SELinux Context
    │
    ▼
container_file_t
    │
    ▼
Container Access Allowed
```

---

# 📌 Task 3: Debug Network Conflicts

---

# 🟢 Subtask 3.1: Check Port Conflicts

```bash
sudo ss -tulnp | grep <port_number>
```

📊 Outcome:
Shows process using the port.

---

## 🔹 Stop Conflicting Container

```bash
podman stop -f <container_id>
```

---

# 🟢 Subtask 3.2: Inspect Network Namespace

## 🔹 View Sandbox Key

```bash
podman inspect <container_id> --format '{{.NetworkSettings.SandboxKey}}'
```

---

## 🔹 Get Container PID

```bash
podman inspect <container_id> --format '{{.State.Pid}}'
```

---

## 🔹 Enter Network Namespace

```bash
sudo nsenter -n -t $(podman inspect --format '{{.State.Pid}}' <container_id>) ip a
```

📊 Outcome:
Shows internal container network interfaces.

---

## 🌐 Network Debug Flow

```text
Host Network
    │
    ▼
Container Namespace
    │
    ▼
nsenter Debug Access
```

---

# 📌 Task 4: Fix Permission Issues

---

# 🟢 Subtask 4.1: Check Container User

```bash
podman inspect <container_id> --format '{{.Config.User}}'
```

📊 Outcome:
Shows running user (root or UID).

---

## 🔹 Run Container as Root (If Needed)

```bash
podman run --user root -it <image> /bin/bash
```

---

# 🟢 Subtask 4.2: Fix Volume Permissions

## 🔹 Use SELinux Relabeling

```bash
podman run -v /host/path:/container/path:Z -it <image> /bin/bash
```

---

## 🧠 Permission Fix Flow

```text
Host Directory
    │
    ▼
SELinux Labeling (:Z)
    │
    ▼
Container Access Granted
```

---

# 🔍 Final Verification

Check container logs:

```bash
podman logs <container_id>
```

---

# 🚨 Troubleshooting Cheat Sheet

---

## ❌ CPU/Memory Issues

✔ Use `podman stats`  
✔ Increase limits  
✔ Optimize application  

---

## ❌ SELinux Denials

✔ Check `ausearch`  
✔ Use `chcon`  
✔ Apply `:Z` for volumes  

---

## ❌ Network Issues

✔ Use `ss -tulnp`  
✔ Check container ports  
✔ Use `nsenter` debugging  

---

## ❌ Permission Issues

✔ Check UID/GID  
✔ Run as root (temporary)  
✔ Fix volume labels  

---

# 🧠 Key Concepts Summary

### 🔹 Resource Limits
Control CPU & memory usage

### 🔹 SELinux Contexts
Security labeling for access control

### 🔹 Network Namespace
Isolated container networking

### 🔹 Volume Permissions
Host-to-container file access rules

---

# 📊 Real-World Use Cases

These skills are used in:

- ☁️ Cloud infrastructure debugging  
- 🚀 CI/CD pipeline troubleshooting  
- 🧠 Microservices failure analysis  
- 🏦 Production incident response  
- 🐳 Kubernetes/OpenShift debugging  

---

# 🧪 Final Checklist

```bash
podman ps
podman stats
podman logs <container_id>
```

Ensure:

✔ No crashes  
✔ No SELinux denials  
✔ No port conflicts  
✔ Proper permissions  

---

# 🧹 Cleanup

```bash
podman stop -a
podman rm -a
```

---

# 🎓 Lab Summary

In this lab, you learned how to:

✅ Diagnose CPU & memory constraints  
✅ Fix SELinux security issues  
✅ Debug network namespace problems  
✅ Resolve permission errors  
✅ Use advanced Podman troubleshooting tools  

---

# 🏆 Completion Badge

🎉 Congratulations!

You have completed:

**Troubleshooting Common Container Issues with Podman**

You are now equipped to handle real-world container failures like a DevOps engineer or SRE.

---

# 🚀 Next Steps

- Learn Kubernetes debugging (`kubectl describe`, `logs`)
- Explore Podman system logs
- Practice OpenShift troubleshooting
- Study observability tools (Prometheus, Grafana)
