# 💾 Backup and Restore Images and Containers with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Backup-Image_Management-blue?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/Restore-Disaster_Recovery-green?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/DevOps-Container_Backup-orange?style=for-the-badge&logo=redhat" />
  <img src="https://img.shields.io/badge/OpenShift-Container_Operations-red?style=for-the-badge&logo=redhatopenshift" />
</p>

---

# 📖 Overview

Container images are valuable assets that often contain application code, configurations, dependencies, and tested environments. Being able to back up, restore, and preserve container states is a critical skill for DevOps engineers, OpenShift administrators, and container platform operators.

In this lab, you will learn how to:

✅ Backup container images using `podman save`

✅ Restore images using `podman load`

✅ Create custom images using `podman commit`

✅ Understand image portability limitations

✅ Prepare images for disaster recovery and migration scenarios

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 Use `podman save` to create image backups

🔹 Use `podman load` to restore images

🔹 Create custom images with `podman commit`

🔹 Understand backup and recovery workflows

🔹 Learn limitations of image backups and commits

🔹 Apply container disaster recovery practices

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement                  | Description                             |
| ---------------------------- | --------------------------------------- |
| 🐳 Podman                    | Version 3.x or later                    |
| 🐧 Linux System              | RHEL 8+, Fedora, Rocky Linux, AlmaLinux |
| 🌐 Internet Access           | Required to pull container images       |
| 💽 Free Disk Space           | At least 1 GB recommended               |
| 📚 Basic Container Knowledge | Familiarity with Podman commands        |

---

# 🏗️ Setup

## 🔹 Verify Podman Installation

```bash
podman --version
```

### Expected Output

```bash
podman version 3.x.x
```

---

## 🔹 Pull a Sample Image

Download Alpine Linux image:

```bash
podman pull docker.io/library/alpine:latest
```

Verify:

```bash
podman images
```

Expected:

```text
docker.io/library/alpine   latest
```

---

# 📌 Task 1: Save an Image to a Tarball

## 🎯 Objective

Learn how to export container images into portable tar files for backup or migration.

---

## 🟢 Subtask 1.1: List Available Images

```bash
podman images
```

Example Output:

```text
REPOSITORY                     TAG
docker.io/library/alpine       latest
```

---

## 🟢 Subtask 1.2: Save Image to Tarball

Create a backup:

```bash
podman save -o alpine_backup.tar docker.io/library/alpine:latest
```

### Command Breakdown

| Option              | Purpose      |
| ------------------- | ------------ |
| `save`              | Export image |
| `-o`                | Output file  |
| `alpine_backup.tar` | Backup file  |
| `alpine:latest`     | Source image |

---

## 🟢 Subtask 1.3: Verify Backup File

Check file size:

```bash
ls -lh alpine_backup.tar
```

Inspect file type:

```bash
file alpine_backup.tar
```

Expected Output:

```text
POSIX tar archive
```

Approximate size:

```text
~3 MB
```

---

## 💡 Why Use podman save?

Useful for:

✅ Air-gapped environments

✅ Offline deployments

✅ Disaster recovery

✅ Image migration

✅ Backup strategies

---

## ⚠️ Troubleshooting

### Error: No Such Image

Verify exact image name:

```bash
podman images
```

---

### Permission Denied

Verify write permissions:

```bash
pwd
ls -ld .
```

---

# 📌 Task 2: Load an Image from Tarball

## 🎯 Objective

Restore a previously saved image backup.

---

## 🟢 Subtask 2.1: Remove Existing Image

```bash
podman rmi docker.io/library/alpine:latest
```

Verify:

```bash
podman images
```

Alpine should no longer appear.

---

## 🟢 Subtask 2.2: Load the Image

Restore image:

```bash
podman load -i alpine_backup.tar
```

Expected Output:

```text
Loaded image(s):
docker.io/library/alpine:latest
```

---

## 🟢 Subtask 2.3: Verify Restored Image

```bash
podman images
```

Expected:

```text
docker.io/library/alpine   latest
```

---

## 💡 Key Concept

`podman load` preserves:

✅ Image layers

✅ Metadata

✅ Tags

✅ Original repository names

---

# 📌 Task 3: Commit Container to New Image

## 🎯 Objective

Create a new image from a modified running container.

---

## 🟢 Subtask 3.1: Start an Interactive Container

```bash
podman run -it \
--name myalpine \
docker.io/library/alpine:latest \
/bin/sh
```

---

## 🟢 Subtask 3.2: Modify Container

Inside the container:

```bash
touch /testfile
```

Add content:

```bash
echo "Lab 12" > /testfile
```

Verify:

```bash
cat /testfile
```

Output:

```text
Lab 12
```

Exit:

```bash
exit
```

---

## 🟢 Subtask 3.3: Commit Changes

Create a new image:

```bash
podman commit myalpine my_custom_alpine:v1
```

Expected Output:

```text
Getting image source signatures
Copying blob
Writing manifest
```

---

## 🟢 Subtask 3.4: Verify New Image

List images:

```bash
podman images
```

Expected:

```text
localhost/my_custom_alpine   v1
```

---

Run new image:

```bash
podman run --rm my_custom_alpine:v1 cat /testfile
```

Expected Output:

```text
Lab 12
```

---

## 💡 Real-World Use Cases

`podman commit` is useful when:

✅ Debugging applications

✅ Capturing runtime changes

✅ Preserving manual configurations

✅ Creating temporary golden images

✅ Testing without Containerfiles

---

# 📌 Task 4: Understanding Limitations

## 🎯 Objective

Learn the practical limitations of save/load and commit operations.

---

## 🟢 Limitation 1: Architecture Dependency

Saved images are architecture-specific.

Example:

```text
x86_64 Image
        ❌
        ↓
ARM System
```

The image may not run correctly.

---

### Verify Architecture

```bash
uname -m
```

Example:

```text
x86_64
```

---

## 🟢 Limitation 2: Commit Does Not Capture Everything

Review documentation:

```bash
podman commit --help | grep -A5 "Limitations"
```

---

### Commit Does NOT Save

❌ Running Processes

❌ Active Network Connections

❌ Volumes

❌ Container Runtime State

❌ External Storage

---

## 🟢 Limitation 3: Storage Consumption

Check disk usage:

```bash
podman system df
```

Example Output:

```text
TYPE           TOTAL     ACTIVE    SIZE
Images         4         2         250MB
Containers     2         1         10MB
```

---

## 💡 Storage Best Practice

Regularly remove:

```bash
podman image prune
```

And:

```bash
podman container prune
```

To reclaim space.

---

# 📊 Backup Workflow Diagram

```text
┌──────────────────────┐
│   Container Image    │
└──────────┬───────────┘
           │
           ▼
    podman save
           │
           ▼
    alpine_backup.tar
           │
           ▼
      Transfer
           │
           ▼
      podman load
           │
           ▼
     Restored Image
```

---

# 📊 Commit Workflow Diagram

```text
Container
    │
    ▼
Modify Files
    │
    ▼
podman commit
    │
    ▼
New Custom Image
    │
    ▼
Reuse Anywhere
```

---

# 🧪 Final Verification

## Verify Saved Backup

```bash
ls -lh alpine_backup.tar
```

---

## Verify Loaded Image

```bash
podman images
```

---

## Verify Custom Image

```bash
podman run --rm my_custom_alpine:v1 cat /testfile
```

Expected:

```text
Lab 12
```

---

## Verify Storage Usage

```bash
podman system df
```

---

# 🚀 Real-World Applications

These techniques are commonly used for:

🔹 Disaster Recovery

🔹 Offline Deployments

🔹 Air-Gapped Systems

🔹 Golden Image Creation

🔹 Development Snapshots

🔹 Container Migration

🔹 Testing and Debugging

🔹 OpenShift Container Management

---

# 📚 Additional Learning

### Podman Save Documentation

```bash
man podman-save
```

### Podman Load Documentation

```bash
man podman-load
```

### Podman Commit Documentation

```bash
man podman-commit
```

### Podman Storage Information

```bash
man podman-system
```

---

# 🧹 Cleanup

## Remove Containers

```bash
podman rm -a
```

---

## Remove Images

```bash
podman rmi -a
```

---

## Remove Backup Archive

```bash
rm alpine_backup.tar
```

---

## Verify Cleanup

```bash
podman images
```

Expected:

```text
No images found
```

---

# 📝 Knowledge Check

### ❓ What is the difference between `podman save` and `podman export`?

### ✅ Answer

```text
podman save
```

Works with images and preserves:

✔ Layers

✔ Metadata

✔ Tags

✔ Repository Information

---

```text
podman export
```

Works with containers and exports:

✔ Container Filesystem Only

❌ No Layers

❌ No Image Metadata

❌ No Tags

---

### ❓ How would you verify the integrity of a saved tarball?

### ✅ Answer

Generate checksum:

```bash
sha256sum alpine_backup.tar
```

Compare checksum after transfer.

---

### ❓ When would you choose `commit` over building with a Containerfile?

### ✅ Answer

Use `commit` when:

✔ Preserving runtime modifications

✔ Debugging environments

✔ No Containerfile exists

✔ Capturing tested container states

---

# 🎓 Lab Summary

In this lab, you successfully learned to:

✅ Backup images using `podman save`

✅ Restore images using `podman load`

✅ Create custom images using `podman commit`

✅ Understand backup and recovery workflows

✅ Learn architecture limitations

✅ Verify storage usage

✅ Apply container disaster recovery concepts

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Backup and Restore Images and Containers with Podman** lab.

You now possess critical image backup, restoration, and disaster recovery skills used in:

🚀 DevOps Engineering

🚀 OpenShift Administration

🚀 Kubernetes Operations

🚀 Enterprise Container Platforms

🚀 Air-Gapped Infrastructure Management

Happy Learning and Happy Containerizing! 🎯
