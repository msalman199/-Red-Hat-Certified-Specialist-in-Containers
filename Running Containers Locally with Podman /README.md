# 🚀 Running Containers Locally with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Linux-Container_Runtime-orange?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/NGINX-Web_Server-green?style=for-the-badge&logo=nginx" />
  <img src="https://img.shields.io/badge/Storage-Volumes-blue?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/DevOps-Container_Management-red?style=for-the-badge&logo=redhat" />
</p>

---

# 📖 Overview

Containers are the foundation of modern cloud-native applications. Podman provides a secure and daemonless way to run, manage, and inspect containers locally.

In this lab, you will learn how to:

✅ Run containers in foreground and detached modes

✅ Configure port mappings between host and containers

✅ Create and use persistent storage volumes

✅ Override container users at runtime

✅ Inspect running containers and monitor resource usage

✅ Perform container cleanup and troubleshooting

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 Run containers in both foreground and detached modes

🔹 Configure port mapping between host and container

🔹 Manage persistent storage using volumes

🔹 Override user settings at runtime

🔹 Inspect running containers and their configurations

🔹 Troubleshoot common container issues

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement            | Description                                    |
| ---------------------- | ---------------------------------------------- |
| 🐳 Podman              | Version 3.0+ Recommended                       |
| 🐧 Linux System        | RHEL, Rocky, AlmaLinux, Fedora, Ubuntu, Debian |
| 🌐 Internet Access     | Required for pulling images                    |
| 💻 Linux CLI Knowledge | Basic command-line experience                  |

---

# 🏗️ Setup Requirements

## 🔹 Install Podman

### RHEL / CentOS / Rocky / Fedora

```bash
sudo dnf install -y podman
```

### Ubuntu / Debian

```bash
sudo apt-get update
sudo apt-get install -y podman
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

# 📌 Task 1: Running Containers in Foreground and Background

---

# 🟢 Subtask 1.1: Run a Container in Foreground

Run an NGINX container:

```bash
podman run --name foreground-container docker.io/library/nginx
```

---

## 💡 What Happens?

This command:

✅ Pulls the image (if not available locally)

✅ Creates a container

✅ Starts NGINX

✅ Attaches your terminal to container output

---

## Expected Outcome

You will see NGINX logs directly in your terminal:

```text
/nginx-entrypoint.sh
Starting nginx...
```

---

## Stop the Container

Press:

```text
CTRL + C
```

to terminate the container.

---

# 🟢 Subtask 1.2: Run a Container in Detached Mode

Run the container in background:

```bash
podman run -d --name background-container docker.io/library/nginx
```

---

## Understanding the Flags

| Flag     | Description           |
| -------- | --------------------- |
| `-d`     | Detached Mode         |
| `--name` | Assign Container Name |

---

## Expected Output

```text
8a9f1f6b3d98c...
```

Container ID will be displayed.

---

## Verify Running Containers

```bash
podman ps
```

Expected:

```text
CONTAINER ID   IMAGE     STATUS
xxxxxxxxxxxx   nginx     Up
```

---

## 📊 Foreground vs Detached Mode

| Mode       | Terminal Attached | Background |
| ---------- | ----------------- | ---------- |
| Foreground | ✅                 | ❌          |
| Detached   | ❌                 | ✅          |

---

# 📌 Task 2: Port Mapping and Volume Binding

---

# 🟢 Subtask 2.1: Bind Container Ports to Host

Run NGINX with port mapping:

```bash
podman run -d \
--name webapp \
-p 8080:80 \
docker.io/library/nginx
```

---

## Port Mapping Explained

```text
HOST PORT      CONTAINER PORT
    8080   ➜        80
```

---

## Network Flow

```text
Browser
   │
   ▼
localhost:8080
   │
   ▼
Container Port 80
   │
   ▼
NGINX
```

---

## Verify Access

```bash
curl http://localhost:8080
```

---

### Expected Output

You should receive the NGINX welcome page:

```html
Welcome to nginx!
```

---

## Verify Port Mapping

```bash
podman port webapp
```

Expected:

```text
80/tcp -> 0.0.0.0:8080
```

---

# 🟢 Subtask 2.2: Create Persistent Storage with Volumes

Create a volume:

```bash
podman volume create mydata
```

---

## Verify Volume

```bash
podman volume ls
```

Expected:

```text
mydata
```

---

## Run Container with Volume

```bash
podman run -d \
--name vol-container \
-v mydata:/data \
docker.io/library/alpine \
tail -f /dev/null
```

---

## Volume Mapping

```text
Named Volume
    │
    ▼
 mydata
    │
    ▼
 /data
```

---

## Verify Mount

```bash
podman exec vol-container ls /data
```

Expected:

```text
(empty directory)
```

---

## Verify Volume Details

```bash
podman inspect vol-container
```

---

# 📌 Task 3: User Management and Runtime Overrides

---

# 🟢 Subtask 3.1: Override User at Runtime

Run container as UID 1000:

```bash
podman run --rm -it \
--user 1000 \
docker.io/library/alpine \
whoami
```

---

### Expected Output

```text
whoami: unknown uid 1000
```

Because UID 1000 doesn't exist inside Alpine.

---

## Run as Known User

```bash
podman run --rm -it \
--user nobody \
docker.io/library/alpine \
whoami
```

---

### Expected Output

```text
nobody
```

---

## Why Use Runtime Users?

Benefits:

✅ Improved Security

✅ Least Privilege Principle

✅ Compliance Requirements

✅ OpenShift Compatibility

---

# 🟢 Subtask 3.2: Inspect Running Containers

Inspect detailed configuration:

```bash
podman inspect webapp
```

---

## Information Available

Inspect shows:

✅ Network Configuration

✅ Mounted Volumes

✅ Environment Variables

✅ User Settings

✅ Container State

---

## View Logs

```bash
podman logs webapp
```

Expected:

```text
NGINX startup logs
```

---

## Monitor Resource Usage

```bash
podman stats
```

Example:

```text
NAME          CPU %     MEM USAGE
webapp        0.10%     15MB
```

---

# 📊 Container Runtime Architecture

```text
┌─────────────────────┐
│     Host System     │
└──────────┬──────────┘
           │
           ▼

     Podman Engine

           │
 ┌─────────┼─────────┐
 │         │         │
 ▼         ▼         ▼

NGINX   Alpine   Volume

 │         │        │
 ▼         ▼        ▼

8080→80  User   Persistent
Mapping  Override Storage
```

---

# 📌 Task 4: Cleanup

---

## Stop All Containers

```bash
podman stop -a
```

---

## Remove Containers

```bash
podman rm -a
```

---

## Remove Volume (Optional)

```bash
podman volume rm mydata
```

---

## Verify Cleanup

```bash
podman ps -a
```

Expected:

```text
No containers found
```

---

# 🔍 Troubleshooting Guide

---

## Problem 1: Permission Denied

Try:

```bash
podman run --privileged IMAGE
```

⚠️ Use only for testing.

---

## Problem 2: Port Already in Use

Check active ports:

```bash
ss -tulnp
```

Filter specific port:

```bash
ss -tulnp | grep 8080
```

---

### Solution

Use another port:

```bash
-p 8081:80
```

---

## Problem 3: Container Exits Immediately

Inspect logs:

```bash
podman logs <container>
```

Example:

```bash
podman logs webapp
```

---

## Problem 4: Volume Not Mounted

Inspect mounts:

```bash
podman inspect vol-container
```

Look for:

```text
Mounts
```

section.

---

# 🧪 Final Verification

## Verify Running Containers

```bash
podman ps
```

---

## Verify Port Mapping

```bash
podman port webapp
```

---

## Verify Volume

```bash
podman volume ls
```

---

## Verify Logs

```bash
podman logs webapp
```

---

## Verify Statistics

```bash
podman stats
```

---

# 🚀 Real-World DevOps Use Cases

Running containers locally is essential for:

🔹 Application Development

🔹 Testing Container Images

🔹 CI/CD Validation

🔹 OpenShift Development

🔹 Kubernetes Troubleshooting

🔹 Container Security Testing

🔹 Local Development Environments

🔹 Infrastructure Automation

---

# 📚 Additional Learning

### Podman Run Documentation

```bash
man podman-run
```

### Podman Inspect Documentation

```bash
man podman-inspect
```

### Podman Stats Documentation

```bash
man podman-stats
```

### Podman Volume Documentation

```bash
man podman-volume
```

### Official Podman Documentation

https://docs.podman.io

---

# 🎓 Lab Summary

In this lab, you successfully learned how to:

✅ Run containers in foreground mode

✅ Run containers in detached mode

✅ Configure host-to-container port mappings

✅ Create and manage persistent volumes

✅ Override users at runtime

✅ Inspect container configurations

✅ Monitor logs and resource usage

✅ Troubleshoot common container issues

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Running Containers Locally with Podman** lab.

You now possess essential container runtime skills used daily by:

🚀 DevOps Engineers

🚀 OpenShift Developers

🚀 Kubernetes Administrators

🚀 Platform Engineers

🚀 Cloud Native Application Teams

These foundational skills prepare you for advanced topics such as:

🔹 Podman Pods

🔹 Rootless Containers

🔹 Kubernetes Workloads

🔹 OpenShift Deployments

🔹 Container Security

Happy Learning and Happy Containerizing! 🎯
