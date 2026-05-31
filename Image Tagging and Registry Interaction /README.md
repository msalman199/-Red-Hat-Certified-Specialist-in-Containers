# 🏷️ Image Tagging and Registry Interaction with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Registry-Docker_Hub-blue?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/Image-Tagging-green?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/DevOps-Container_Management-orange?style=for-the-badge&logo=redhat" />
  <img src="https://img.shields.io/badge/OpenShift-Registry_Ready-red?style=for-the-badge&logo=redhatopenshift" />
</p>

---

# 📖 Overview

Container registries are the backbone of modern containerized application delivery. They allow developers and DevOps engineers to store, share, version, and deploy container images efficiently.

In this lab, you will learn how to:

✅ Tag container images using semantic versioning

✅ Authenticate with container registries

✅ Push images to Docker Hub or private registries

✅ Pull images from registries

✅ Understand image digests and tag immutability

✅ Apply production-ready image management practices

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 Tag container images with semantic versioning

🔹 Authenticate with container registries (Docker Hub or Private Registry)

🔹 Push and pull images from registries

🔹 Understand image digests and immutable image references

🔹 Manage image versions effectively

🔹 Follow container image best practices

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement           | Description                       |
| --------------------- | --------------------------------- |
| 🐳 Podman             | Version 3.0+ Recommended          |
| 👤 Docker Hub Account | Or access to a private registry   |
| 🐧 Linux Knowledge    | Basic command-line skills         |
| 🌐 Internet Access    | Required for registry interaction |

---

# 🏗️ Lab Setup

## 🔹 Verify Podman Installation

```bash
podman --version
```

### Expected Output

```bash
podman version 3.x.x
```

---

## 🔹 Verify Internet Connectivity

```bash
ping -c 4 docker.io
```

Expected:

```text
Successful replies from Docker Hub
```

---

# 💡 Understanding Container Image Naming

Container images follow this structure:

```text
REGISTRY/USERNAME/IMAGE_NAME:TAG
```

Example:

```text
docker.io/msalman199/my-nginx:v1.0.0
```

Breakdown:

| Component  | Example    |
| ---------- | ---------- |
| Registry   | docker.io  |
| Username   | msalman199 |
| Image Name | my-nginx   |
| Tag        | v1.0.0     |

---

# 📌 Task 1: Tagging Images Semantically

## 🎯 Objective

Learn to apply meaningful tags to container images.

---

## 🟢 Subtask 1.1: List Existing Images

```bash
podman images
```

Example Output:

```text
REPOSITORY                     TAG
docker.io/library/nginx       latest
```

---

## 🟢 Subtask 1.2: Tag an Existing Image

General Syntax:

```bash
podman tag <source_image> <new_name>:<tag>
```

Example:

```bash
podman tag docker.io/library/nginx my-nginx:v1.0
```

---

## 🟢 Subtask 1.3: Verify New Tag

```bash
podman images | grep nginx
```

Expected Output:

```text
docker.io/library/nginx      latest
localhost/my-nginx           v1.0
```

---

## 💡 Semantic Versioning (SemVer)

Production images should use:

```text
vMAJOR.MINOR.PATCH
```

Examples:

```text
v1.0.0
v1.1.0
v1.1.1
v2.0.0
```

### Version Meaning

| Version Part | Description      |
| ------------ | ---------------- |
| MAJOR        | Breaking Changes |
| MINOR        | New Features     |
| PATCH        | Bug Fixes        |

Example:

```text
v2.3.5
```

Means:

```text
Major Version = 2
Minor Features = 3
Patch Fixes = 5
```

---

# 📌 Task 2: Logging into a Registry

## 🎯 Objective

Authenticate with Docker Hub or a private registry.

---

## 🟢 Subtask 2.1: Login to Docker Hub

```bash
podman login docker.io
```

You will be prompted for:

```text
Username:
Password:
```

---

### Expected Output

```text
Login Succeeded!
```

---

## 🟢 Subtask 2.2: Verify Authentication

```bash
podman info | grep -A 5 "registries"
```

Expected:

```text
Registry information displayed
```

---

## ⚠️ Troubleshooting Tip

For private registries:

```bash
podman login registry.example.com
```

Replace:

```text
registry.example.com
```

With your registry URL.

---

# 📌 Task 3: Pushing Images to a Registry

## 🎯 Objective

Upload your image to Docker Hub or another registry.

---

## 🟢 Subtask 3.1: Tag Image with Registry Prefix

Replace:

```text
<your_username>
```

With your Docker Hub username.

Example:

```bash
podman tag my-nginx:v1.0 docker.io/<your_username>/my-nginx:v1.0
```

Example:

```bash
podman tag my-nginx:v1.0 docker.io/msalman199/my-nginx:v1.0
```

---

## 🟢 Subtask 3.2: Push Image

```bash
podman push docker.io/<your_username>/my-nginx:v1.0
```

Example:

```bash
podman push docker.io/msalman199/my-nginx:v1.0
```

---

### Expected Output

```text
Writing manifest...
Storing signatures...
Copying blobs...
```

---

## 🟢 Subtask 3.3: Verify on Docker Hub

Visit:

```text
https://hub.docker.com
```

Steps:

✅ Login

✅ Open Repositories

✅ Verify uploaded image

---

### Expected Outcome

```text
my-nginx:v1.0 visible in repository
```

---

# 📌 Task 4: Pulling Images from a Registry

## 🎯 Objective

Download images from a registry.

---

## 🟢 Subtask 4.1: Remove Local Image

```bash
podman rmi docker.io/<your_username>/my-nginx:v1.0
```

Example:

```bash
podman rmi docker.io/msalman199/my-nginx:v1.0
```

---

## 🟢 Subtask 4.2: Pull Image

```bash
podman pull docker.io/<your_username>/my-nginx:v1.0
```

Example:

```bash
podman pull docker.io/msalman199/my-nginx:v1.0
```

---

### Expected Output

```text
Trying to pull...
Downloading layers...
```

---

## 🟢 Subtask 4.3: Verify Download

```bash
podman images
```

Expected:

```text
docker.io/msalman199/my-nginx   v1.0
```

---

## 💡 Key Concept

Pulling an image retrieves:

```text
Registry → Local System
```

The image associated with the tag at that moment is downloaded.

---

# 📌 Task 5: Understanding Tag Immutability and Digests

## 🎯 Objective

Learn about image digests and tag behavior.

---

## 🟢 Subtask 5.1: Inspect Image Digest

```bash
podman inspect --format '{{.Digest}}' docker.io/<your_username>/my-nginx:v1.0
```

Example:

```bash
podman inspect --format '{{.Digest}}' docker.io/msalman199/my-nginx:v1.0
```

Expected:

```text
sha256:3f5f9c4c6f7a9d...
```

---

## 🟢 Subtask 5.2: Pull by Digest

Syntax:

```bash
podman pull docker.io/<your_username>/my-nginx@<digest>
```

Example:

```bash
podman pull docker.io/msalman199/my-nginx@sha256:3f5f9c4c6f7a9d...
```

---

## 💡 Why Digests Matter

A digest:

✅ Is immutable

✅ Always points to the exact image

✅ Cannot be overwritten

✅ Ensures deployment consistency

---

## 🟢 Subtask 5.3: Test Tag Immutability

Push a newer image:

```bash
podman push docker.io/<your_username>/my-nginx:v1.0
```

Observe:

```text
Tag remains the same
Digest changes
```

---

## 📊 Tags vs Digests

| Feature                 | Tag | Digest |
| ----------------------- | --- | ------ |
| Human Friendly          | ✅   | ❌      |
| Mutable                 | ✅   | ❌      |
| Immutable               | ❌   | ✅      |
| Production Safe         | ⚠️  | ✅      |
| Exact Version Reference | ❌   | ✅      |

---

## 📦 Example

```text
docker.io/msalman199/my-nginx:v1.0
```

May point to:

```text
sha256:111111
```

Later:

```text
sha256:222222
```

Tag remains:

```text
v1.0
```

Digest changes.

---

# 🔍 Troubleshooting Guide

## Problem 1: Login Failed

Verify credentials:

```bash
podman logout docker.io
```

Then:

```bash
podman login docker.io
```

---

## Problem 2: Push Denied

Ensure image name contains your username:

```text
docker.io/<your_username>/image
```

Incorrect:

```text
docker.io/library/image
```

---

## Problem 3: Pull Fails

Verify image exists:

```bash
podman search my-nginx
```

---

## Problem 4: Digest Missing

Inspect image:

```bash
podman inspect IMAGE
```

Or:

```bash
podman image inspect IMAGE
```

---

# 🧪 Final Verification

## List Images

```bash
podman images
```

---

## Verify Registry Login

```bash
podman login docker.io
```

---

## Verify Push

```bash
podman push docker.io/<your_username>/my-nginx:v1.0
```

---

## Verify Pull

```bash
podman pull docker.io/<your_username>/my-nginx:v1.0
```

---

## Verify Digest

```bash
podman inspect --format '{{.Digest}}' docker.io/<your_username>/my-nginx:v1.0
```

---

# 🚀 Real-World DevOps Use Cases

Container registries are used for:

🔹 CI/CD Pipelines

🔹 Kubernetes Deployments

🔹 OpenShift Image Streams

🔹 GitOps Workflows

🔹 Multi-Environment Deployments

🔹 Image Promotion Strategies

🔹 Production Release Management

---

# 📚 Additional Learning

### Podman Registry Commands

```bash
man podman-login
```

```bash
man podman-push
```

```bash
man podman-pull
```

---

### Docker Hub

https://hub.docker.com

### Podman Documentation

https://docs.podman.io

### OpenShift Image Management

https://docs.openshift.com

---

# 🧹 Cleanup

⚠️ Warning: This removes ALL local images.

```bash
podman rmi -f $(podman images -q)
```

Verify:

```bash
podman images
```

Expected:

```text
No images found
```

---

# 🎓 Lab Summary

In this lab, you successfully learned to:

✅ Tag images using semantic versioning

✅ Authenticate with container registries

✅ Push images to Docker Hub

✅ Pull images from registries

✅ Understand image digests

✅ Work with immutable image references

✅ Apply production-ready image versioning practices

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Image Tagging and Registry Interaction with Podman** lab.

You now possess essential container image management skills used in:

🚀 DevOps Engineering

🚀 Kubernetes Administration

🚀 OpenShift Development

🚀 CI/CD Pipeline Automation

🚀 Enterprise Container Platforms

Happy Learning and Happy Containerizing! 🎯
