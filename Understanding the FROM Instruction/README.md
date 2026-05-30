# 🐳 Understanding the FROM Instruction 

<div align="center">

# 🚀 Understanding the FROM Instruction

### Learn the Foundation of Container Image Creation with Podman

![Podman](https://img.shields.io/badge/Podman-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Containerfile](https://img.shields.io/badge/Containerfile-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![RedHat UBI](https://img.shields.io/badge/Red_Hat-EE0000?style=for-the-badge&logo=redhat&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Containers](https://img.shields.io/badge/Containers-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![DevOps](https://img.shields.io/badge/DevOps-0A66C2?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

# 📖 Overview

The **FROM** instruction is the most important instruction in a Containerfile (or Dockerfile). Every container image begins with a base image, and the FROM instruction defines that starting point.

In this lab, you will learn how to:

- 🔹 Understand the purpose of the FROM instruction
- 🔹 Select appropriate base images
- 🔹 Pin image versions for reproducible builds
- 🔹 Create a simple Containerfile
- 🔹 Build and verify a container image

---

# 🎯 Objectives

By the end of this lab, you will be able to:

- ✅ Understand the purpose of the FROM instruction
- ✅ Explain why base images are important
- ✅ Select appropriate container base images
- ✅ Pin image versions for reproducibility
- ✅ Create a simple Containerfile
- ✅ Build container images using Podman

---

# 📋 Prerequisites

Before starting this lab, ensure you have:

| Requirement | Description |
|------------|------------|
| 🖥️ Linux Knowledge | Basic command-line skills |
| 🐳 Podman | Version 3.0+ recommended |
| 🌐 Internet Access | Required for pulling images |
| 📦 Registry Access | Public image registry access |

---

# 🏗️ Container Build Architecture

```text
┌───────────────────────┐
│      Base Image       │
│    (UBI 8.7 Image)    │
└───────────┬───────────┘
            │
            ▼

┌───────────────────────┐
│  FROM Instruction     │
└───────────┬───────────┘
            │
            ▼

┌───────────────────────┐
│ Additional Layers     │
│ RUN, COPY, ENV, etc.  │
└───────────┬───────────┘
            │
            ▼

┌───────────────────────┐
│ Final Container Image │
└───────────────────────┘
```

---

# ⚙️ Setup Requirements

## 🔍 Verify Podman Installation

Run:

```bash
podman --version
```

### Expected Output

```bash
podman version 3.4.4
```

---

## 📁 Create Working Directory

```bash
mkdir from-lab
cd from-lab
```

---

# 🛠️ Task 1: Understanding the FROM Instruction

## 📌 Subtask 1.1: What is the FROM Instruction?

The FROM instruction:

✅ Is usually the first instruction in a Containerfile

✅ Defines the base image

✅ Creates the starting layer for image builds

✅ Can reference local or remote images

---

### Example

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.7
```

This tells Podman:

> Start building the image using Red Hat UBI 8.7 as the base layer.

---

## 📚 Why is FROM Important?

Every container image inherits:

- Operating System Components
- Libraries
- Packages
- Security Updates
- Runtime Environment

from its base image.

Without a FROM instruction, Podman does not know how to begin building your image.

---

# 📌 Subtask 1.2: Importance of Base Images

Selecting the correct base image is critical.

---

## 📦 Size Considerations

### Lightweight Images

Examples:

- Alpine Linux
- BusyBox

Benefits:

✅ Smaller image size

✅ Faster downloads

✅ Reduced attack surface

---

### Full Distribution Images

Examples:

- Ubuntu
- Debian
- Red Hat UBI

Benefits:

✅ More tools included

✅ Easier troubleshooting

✅ Better package availability

---

## 🔒 Security Considerations

Choose:

✅ Official Images

✅ Vendor-Supported Images

✅ Frequently Updated Images

Avoid:

❌ Untrusted Community Images

❌ Unmaintained Images

---

## 🔄 Maintenance Considerations

Select images that:

- Receive security updates
- Have long-term support
- Are actively maintained

---

## 💻 Compatibility Considerations

Verify:

- CPU Architecture
- Operating System
- Application Requirements

---

# 🚀 Task 2: Selecting and Pinning a Base Image

## 📌 Subtask 2.1: Find Official Images

Search Red Hat Universal Base Images:

```bash
podman search registry.access.redhat.com/ubi8
```

### Expected Output

```text
NAME
registry.access.redhat.com/ubi8/ubi
registry.access.redhat.com/ubi8/python-39
registry.access.redhat.com/ubi8/nodejs-16
...
```

---

## 🔍 Examine Image Metadata

Install Skopeo (if needed):

```bash
sudo dnf install skopeo
```

Inspect image information:

```bash
skopeo inspect docker://registry.access.redhat.com/ubi8/ubi:latest
```

---

### Information Displayed

- Image Digest
- Architecture
- Layers
- Labels
- Tags

---

# 📌 Subtask 2.2: Pin Image Versions

## ⚠️ Avoid Using latest

Bad Practice:

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi:latest
```

Reason:

❌ Image may change unexpectedly

❌ Builds become inconsistent

---

## ✅ Use Version Tags

Recommended:

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.7
```

---

## 🔒 Use Digests for Maximum Reproducibility

Example:

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi@sha256:<digest>
```

Benefits:

✅ Exact image version

✅ Fully reproducible builds

✅ Supply chain security

---

## Pull a Specific Version

```bash
podman pull registry.access.redhat.com/ubi8/ubi:8.7
```

---

# 🛠️ Task 3: Writing a Simple Containerfile

## 📌 Subtask 3.1: Create a Containerfile

Create:

```bash
touch Containerfile
```

---

## ✍️ Add Content

```dockerfile
# Use a pinned UBI image

FROM registry.access.redhat.com/ubi8/ubi:8.7

# Image metadata

LABEL maintainer="your.email@example.com"

# Create a status file

RUN echo "Base image successfully set up" > /tmp/status.txt
```

---

# 📚 Understanding the Containerfile

## FROM

Defines the base image.

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.7
```

---

## LABEL

Adds metadata.

```dockerfile
LABEL maintainer="your.email@example.com"
```

---

## RUN

Executes commands during build.

```dockerfile
RUN echo "Base image successfully set up" > /tmp/status.txt
```

---

# 🚀 Subtask 3.2: Build the Container Image

Build:

```bash
podman build -t my-base-image .
```

---

### Expected Output

```text
Successfully tagged my-base-image:latest
```

---

# 🔍 Verify the Image

List images:

```bash
podman images
```

### Expected Output

```text
REPOSITORY       TAG       IMAGE ID
my-base-image    latest    abc123xyz
```

---

# 🧪 Test the Image

Verify the status file:

```bash
podman run --rm my-base-image cat /tmp/status.txt
```

### Expected Output

```text
Base image successfully set up
```

---

# 🏗️ Build Workflow

```text
Create Containerfile
          │
          ▼

Define Base Image
      (FROM)
          │
          ▼

Add Instructions
   (LABEL, RUN)
          │
          ▼

Build Image
 (podman build)
          │
          ▼

Verify Image
 (podman run)
```

---

# 🚨 Troubleshooting Tips

## ⚠️ Image Not Found

Verify:

```bash
podman search ubi8
```

Check:

- Internet Connectivity
- Registry URL
- Image Tag

---

## ⚠️ Build Failures

Check:

```bash
cat Containerfile
```

Verify:

- Correct Syntax
- Valid Instructions
- Proper Indentation

---

### Rebuild Without Cache

```bash
podman build --no-cache -t my-base-image .
```

---

## ⚠️ Permission Issues

Use:

```bash
podman info
```

Verify:

- Rootless Configuration
- User Permissions
- SELinux Policies

---

## ⚠️ Registry Authentication Problems

Login:

```bash
podman login registry.access.redhat.com
```

---

# 🔥 Best Practices for FROM

| Practice | Recommendation |
|-----------|---------------|
| Version Pinning | ✅ Always |
| Use latest Tag | ❌ Avoid |
| Official Images | ✅ Use |
| Vendor Supported Images | ✅ Prefer |
| Image Digest | ✅ Best Option |
| Small Images | ✅ When Possible |

---

# 🎉 Conclusion

In this lab, you successfully:

✅ Learned the role of the FROM instruction

✅ Explored container base images

✅ Selected and pinned image versions

✅ Created a Containerfile

✅ Built a container image using Podman

✅ Verified image functionality

Understanding the FROM instruction is essential for creating reliable and reproducible container builds. Properly selecting and pinning base images ensures consistent behavior across development, testing, and production environments.

---

# 🚀 Next Steps

Continue your containerization journey:

- 🔹 Explore Alpine Linux Images
- 🔹 Build Multi-Stage Containers
- 🔹 Learn COPY and ADD Instructions
- 🔹 Work with ENV Variables
- 🔹 Study Image Layering
- 🔹 Analyze Images Using podman history
- 🔹 Implement Secure Container Practices

---

# 🧹 Cleanup (Optional)

## Remove Created Image

```bash
podman rmi my-base-image
```

---

## Remove Unused Images

```bash
podman image prune -a
```

---

# 📚 Additional Resources

### Podman Documentation

https://docs.podman.io/

### Red Hat UBI Images

https://catalog.redhat.com/software/containers/search

### Containerfile Reference

https://docs.docker.com/engine/reference/builder/

---

<div align="center">
### 🐳 Mastering the FROM Instruction!

**Happy Building & Happy Learning! 🚀**

</div>
