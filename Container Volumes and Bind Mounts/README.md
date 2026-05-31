# 💾 Container Volumes and Bind Mounts with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Linux-RHEL-red?style=for-the-badge&logo=redhat" />
  <img src="https://img.shields.io/badge/Storage-Volumes-blue?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/SELinux-Security-orange?style=for-the-badge&logo=redhat" />
  <img src="https://img.shields.io/badge/OpenShift-Container_Storage-red?style=for-the-badge&logo=redhatopenshift" />
</p>

---

# 📖 Overview

Containers are designed to be ephemeral, meaning data inside a container can be lost when the container is removed. To solve this challenge, Podman provides **Named Volumes** and **Bind Mounts** for persistent storage.

This lab demonstrates how to:

✅ Create and manage named volumes

✅ Implement bind mounts

✅ Configure SELinux contexts

✅ Manage host directory permissions

✅ Verify persistent data storage

✅ Troubleshoot storage-related issues

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 Understand different volume types in containerization

🔹 Create and manage named volumes

🔹 Implement bind mounts with proper SELinux context (`:Z`)

🔹 Configure host directory permissions for container access

🔹 Verify data persistence across container lifecycles

🔹 Troubleshoot common storage issues

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement        | Description                         |
| ------------------ | ----------------------------------- |
| 🐳 Podman          | Version 3.0+ Recommended            |
| 🐧 Linux Knowledge | Basic command-line experience       |
| 🔒 SELinux         | Enabled System (RHEL/Fedora/CentOS) |
| 👨‍💻 sudo Access  | Required for some operations        |

---

# 🏗️ Lab Setup

## 🔹 Install Podman

```bash
sudo dnf install podman -y
```

---

## 🔹 Verify Installation

```bash
podman --version
```

### Expected Output

```bash
podman version 3.x.x
```

---

# 💡 Understanding Container Storage

## 📦 Named Volumes

Managed entirely by Podman.

### Advantages

✅ Persistent

✅ Portable

✅ Container-independent

✅ Recommended for production workloads

---

## 📂 Bind Mounts

Maps a host directory directly into a container.

### Advantages

✅ Easy access from host

✅ Ideal for development

✅ Real-time file synchronization

---

## 📊 Storage Comparison

| Feature                | Named Volume | Bind Mount  |
| ---------------------- | ------------ | ----------- |
| Managed by Podman      | ✅            | ❌           |
| Persistent             | ✅            | ✅           |
| Direct Host Access     | ❌            | ✅           |
| SELinux Considerations | Minimal      | Required    |
| Production Use         | Recommended  | Situational |

---

# 📌 Task 1: Creating and Using Named Volumes

---

## 🟢 Subtask 1.1: Create a Named Volume

Create a volume:

```bash
podman volume create mydata_volume
```

### Explanation

This creates a persistent storage volume managed by Podman.

---

### Verify Volume Creation

```bash
podman volume ls
```

### Expected Output

```text
DRIVER      VOLUME NAME
local       mydata_volume
```

---

## 🟢 Subtask 1.2: Use the Volume in a Container

Run an Alpine container:

```bash
podman run -d \
--name volume_test \
-v mydata_volume:/data \
docker.io/library/alpine \
sleep infinity
```

### Explanation

```text
mydata_volume  --->  /data
```

The volume is mounted inside the container at `/data`.

---

### Verify Mount

```bash
podman exec volume_test ls /data
```

Expected:

```text
(No output)
```

Directory is currently empty.

---

## 🟢 Subtask 1.3: Test Data Persistence

Create a file:

```bash
podman exec volume_test sh -c "echo 'Persistent Data' > /data/testfile"
```

---

Stop and remove container:

```bash
podman stop volume_test
podman rm volume_test
```

---

Create a new container using the same volume:

```bash
podman run \
--name new_test \
-v mydata_volume:/data \
docker.io/library/alpine \
cat /data/testfile
```

### Expected Output

```text
Persistent Data
```

✅ Data survives container deletion.

---

# 📌 Task 2: Bind Mounts with SELinux Context

---

## 🟢 Subtask 2.1: Create Host Directory

```bash
mkdir ~/container_data
```

Create a file:

```bash
echo "Host file content" > ~/container_data/hostfile.txt
```

---

### Verify File

```bash
cat ~/container_data/hostfile.txt
```

Output:

```text
Host file content
```

---

## 🟢 Subtask 2.2: Mount with :Z Option

Run:

```bash
podman run -it --rm \
-v ~/container_data:/container_data:Z \
docker.io/library/alpine \
cat /container_data/hostfile.txt
```

### Explanation

The `:Z` option:

✅ Relabels content for container access

✅ Applies correct SELinux context

✅ Prevents permission denials

---

### Expected Output

```text
Host file content
```

---

## 🟢 Subtask 2.3: Verify SELinux Context

Check labels:

```bash
ls -Z ~/container_data/hostfile.txt
```

Expected Output:

```text
unconfined_u:object_r:container_file_t:s0
```

---

# 📌 Task 3: Adjusting Host Directory Permissions

---

## 🟢 Subtask 3.1: Create Restricted Directory

```bash
sudo mkdir /restricted_data
```

```bash
sudo chmod 700 /restricted_data
```

```bash
sudo chown root:root /restricted_data
```

---

### Verify

```bash
ls -ld /restricted_data
```

Expected:

```text
drwx------ root root
```

---

## 🟢 Subtask 3.2: Test Access Without Adjustments

Run:

```bash
podman run -it --rm \
-v /restricted_data:/data \
docker.io/library/alpine \
touch /data/testfile
```

### Expected Result

```text
Permission denied
```

---

## 🟢 Subtask 3.3: Fix Permissions and SELinux

Modify permissions:

```bash
sudo chmod 755 /restricted_data
```

Apply SELinux label:

```bash
sudo chcon -t container_file_t /restricted_data
```

Run again:

```bash
podman run -it --rm \
-v /restricted_data:/data:Z \
docker.io/library/alpine \
touch /data/testfile
```

---

### Verification

```bash
ls -lZ /restricted_data
```

Expected:

```text
-rw-r--r-- container_file_t testfile
```

---

# 📌 Task 4: Comprehensive Data Persistence Test

---

## 🟢 Subtask 4.1: Create Test Environment

```bash
podman run -d \
--name persist_test \
-v mydata_volume:/data \
-v ~/container_data:/container_data:Z \
docker.io/library/alpine \
sleep infinity
```

---

## 🟢 Subtask 4.2: Write Test Data

### Write to Named Volume

```bash
podman exec persist_test \
sh -c "echo 'Named Volume Data' >> /data/named.txt"
```

---

### Write to Bind Mount

```bash
podman exec persist_test \
sh -c "echo 'Bind Mount Data' >> /container_data/bind.txt"
```

---

## 🟢 Subtask 4.3: Verify Persistence

Stop and remove:

```bash
podman stop persist_test
```

```bash
podman rm persist_test
```

---

Verify Named Volume:

```bash
podman run --rm \
-v mydata_volume:/data \
docker.io/library/alpine \
cat /data/named.txt
```

Expected:

```text
Named Volume Data
```

---

Verify Bind Mount:

```bash
cat ~/container_data/bind.txt
```

Expected:

```text
Bind Mount Data
```

---

# 🔍 Troubleshooting Guide

---

## Problem 1: Permission Denied

Check:

```bash
ls -lZ
```

Verify:

```bash
getenforce
```

Expected:

```text
Enforcing
```

---

### Solution

Use:

```bash
:Z
```

Example:

```bash
-v ~/container_data:/container_data:Z
```

---

## Problem 2: Volume Not Persisting

Check:

```bash
podman volume ls
```

Inspect:

```bash
podman inspect <container>
```

Verify mount points:

```bash
podman inspect volume_test
```

---

## Problem 3: SELinux Issues

Search audit logs:

```bash
sudo ausearch -m avc -ts recent
```

---

Temporary testing mode:

```bash
sudo setenforce 0
```

Re-enable:

```bash
sudo setenforce 1
```

---

# 📊 Volume Architecture

```text
┌─────────────────────────┐
│      Host System        │
└──────────┬──────────────┘
           │
 ┌─────────┴─────────┐
 │                   │
 ▼                   ▼

Named Volume      Bind Mount
mydata_volume     ~/container_data

 │                   │
 └──────┬────────────┘
        ▼

   Container
      /data
 /container_data
```

---

# 🧪 Final Verification

---

### List Volumes

```bash
podman volume ls
```

---

### Verify Named Volume

```bash
podman run --rm \
-v mydata_volume:/data \
docker.io/library/alpine \
ls /data
```

---

### Verify Bind Mount

```bash
ls ~/container_data
```

---

### Verify SELinux Context

```bash
ls -Z ~/container_data
```

---

# 🧹 Cleanup

Remove volume:

```bash
podman volume rm mydata_volume
```

---

Remove all containers:

```bash
podman rm -f $(podman ps -aq)
```

---

Remove bind mount directory:

```bash
rm -rf ~/container_data
```

---

Remove restricted directory:

```bash
sudo rm -rf /restricted_data
```

---

# 🚀 Real-World DevOps Use Cases

Container storage is heavily used for:

🔹 Databases

🔹 Application Logs

🔹 Kubernetes Persistent Volumes

🔹 OpenShift Storage Classes

🔹 CI/CD Artifacts

🔹 Backup Solutions

🔹 Shared Application Data

---

# 📚 Additional Resources

### Podman Volume Documentation

```bash
man podman-volume
```

### SELinux for Containers

```bash
man container_selinux
```

### Red Hat Container Storage Guide

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/building_running_and_managing_containers/index

---

# 🎓 Lab Summary

In this lab, you successfully learned to:

✅ Create and manage named volumes

✅ Configure bind mounts

✅ Apply SELinux contexts using `:Z`

✅ Manage host directory permissions

✅ Verify persistent storage

✅ Troubleshoot volume and SELinux issues

✅ Understand production container storage concepts

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Container Volumes and Bind Mounts with Podman** lab.

You now possess foundational container storage skills used in:

🚀 DevOps Engineering

🚀 OpenShift Administration

🚀 Kubernetes Storage Management

🚀 Red Hat Enterprise Linux Container Platforms

Happy Learning and Happy Containerizing! 🎯
