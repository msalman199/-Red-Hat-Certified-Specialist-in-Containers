# 🔍 Inspecting Containers and Images with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Inspection-Metadata-blue?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/Debugging-Troubleshooting-green?style=for-the-badge&logo=gnu-bash" />
  <img src="https://img.shields.io/badge/Containers-Analysis-orange?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/DevOps-Podman-red?style=for-the-badge&logo=redhat" />
</p>

---

# 📖 Overview

Container inspection is one of the most important troubleshooting and operational skills in containerized environments. Podman provides the powerful `podman inspect` command, which allows administrators and developers to view detailed metadata about containers and images.

In this lab, you will learn how to:

✅ Inspect container metadata

✅ Inspect image metadata

✅ Extract environment variables

✅ Analyze volume mounts

✅ Review network port mappings

✅ Troubleshoot containers using state and exit codes

---

# 🎯 Lab Objectives

By the end of this lab, you will be able to:

🔹 Use `podman inspect` to examine container and image metadata

🔹 Extract and analyze environment variables

🔹 Review volume mounts and network port configurations

🔹 Interpret container status and exit codes

🔹 Troubleshoot container runtime issues

🔹 Use formatted output for targeted inspections

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement              | Description                            |
| ------------------------ | -------------------------------------- |
| 🐳 Podman                | Installed and configured               |
| 🐧 Linux System          | RHEL, CentOS, Fedora, Rocky, AlmaLinux |
| 💻 Terminal Access       | Bash Shell                             |
| 🌐 Internet Access       | To pull container images               |
| 📚 Basic Linux Knowledge | Command-line familiarity               |

---

# 🏗️ Lab Setup

## 🔹 Install Podman

### RHEL / CentOS / Fedora

```bash
sudo dnf install -y podman
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

## 🔹 Pull Sample Image

```bash
podman pull docker.io/library/nginx:alpine
```

Verify:

```bash
podman images
```

Expected:

```text
docker.io/library/nginx   alpine
```

---

# 📌 Task 1: Inspecting Container and Image Metadata

---

# 🟢 Subtask 1.1: Basic Inspection

## Run a Sample Container

```bash
podman run -d \
--name my_nginx \
-p 8080:80 \
nginx:alpine
```

---

## Inspect the Container

```bash
podman inspect my_nginx
```

---

### Expected Output

```json
{
  "Id": "...",
  "Created": "...",
  "State": {},
  "Config": {},
  "NetworkSettings": {}
}
```

The output is returned as JSON containing complete runtime information.

---

## Inspect the Image

```bash
podman inspect nginx:alpine
```

---

## 💡 Key Concept

### Image Inspection

Shows:

✅ Image Layers

✅ Build Information

✅ Default Environment Variables

✅ Entrypoint Configuration

✅ Labels

---

### Container Inspection

Shows:

✅ Runtime State

✅ Current Network Settings

✅ Volume Mounts

✅ Running Processes

✅ Exit Status

---

# 🟢 Subtask 1.2: Filtering Specific Information

---

## Get Container IP Address

```bash
podman inspect my_nginx \
--format '{{.NetworkSettings.IPAddress}}'
```

Expected:

```text
10.x.x.x
```

---

## Get Container Creation Time

```bash
podman inspect my_nginx \
--format '{{.Created}}'
```

Expected:

```text
2025-05-30T10:30:00Z
```

---

## Why Use --format?

Benefits:

✅ Faster output

✅ Easier automation

✅ Scripting friendly

✅ Extract specific values

---

# 📌 Task 2: Extracting Environment Variables

---

# 🟢 Subtask 2.1: Viewing All Environment Variables

Run a container with custom variables:

```bash
podman run -d \
--name env_test \
-e APP_COLOR=blue \
-e APP_MODE=prod \
nginx:alpine
```

---

## Inspect Environment Variables

```bash
podman inspect env_test \
--format '{{.Config.Env}}'
```

Expected:

```text
[APP_COLOR=blue APP_MODE=prod]
```

---

## Common Environment Variables

| Variable  | Purpose                |
| --------- | ---------------------- |
| APP_COLOR | Application theme      |
| APP_MODE  | Runtime mode           |
| PATH      | System executable path |
| HOSTNAME  | Container hostname     |

---

# 🟢 Subtask 2.2: Finding Specific Variables

Extract only APP_COLOR:

```bash
podman inspect env_test \
--format '{{range .Config.Env}}{{if eq (index (split . "=") 0) "APP_COLOR"}}{{.}}{{end}}{{end}}'
```

Expected Output:

```text
APP_COLOR=blue
```

---

## Environment Variable Flow

```text
Container Start
        │
        ▼
 Environment Variables
        │
        ▼
 Application Configuration
        │
        ▼
 Runtime Behavior
```

---

# 📌 Task 3: Reviewing Volume Mounts and Ports

---

# 🟢 Subtask 3.1: Examining Volume Mounts

Create a container with a mounted volume:

```bash
podman run -d \
--name vol_test \
-v /tmp:/container_tmp \
nginx:alpine
```

---

## Inspect Mount Information

```bash
podman inspect vol_test \
--format '{{.Mounts}}'
```

Expected:

```text
[/tmp:/container_tmp]
```

---

## Volume Mapping Visualization

```text
Host System
    │
    ▼
 /tmp
    │
    ▼
Container
    │
    ▼
/container_tmp
```

---

## Why Volume Mounts Matter

✅ Persistent Data

✅ Configuration Sharing

✅ Log Storage

✅ Backup Integration

---

# 🟢 Subtask 3.2: Analyzing Port Mappings

Inspect port bindings:

```bash
podman inspect my_nginx \
--format '{{.NetworkSettings.Ports}}'
```

Expected:

```text
80/tcp -> 8080
```

---

## Extract Host Port Only

```bash
podman inspect my_nginx \
--format '{{(index (index .NetworkSettings.Ports "80/tcp") 0).HostPort}}'
```

Expected:

```text
8080
```

---

## Port Mapping Flow

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
NGINX Service
```

---

# 📌 Task 4: Analyzing Container Status and Exit Codes

---

# 🟢 Subtask 4.1: Checking Running Status

Create a failing container:

```bash
podman run \
--name fail_test \
alpine \
sh -c "exit 3"
```

---

## Inspect Exit Code

```bash
podman inspect fail_test \
--format '{{.State.ExitCode}}'
```

Expected:

```text
3
```

---

## Understanding Exit Codes

| Exit Code | Meaning            |
| --------- | ------------------ |
| 0         | Success            |
| 1         | General Error      |
| 2         | Misuse of Command  |
| 126       | Permission Problem |
| 127       | Command Not Found  |
| >0        | Application Error  |

---

## Troubleshooting Tip

Non-zero exit codes indicate failures.

Check:

```bash
podman logs fail_test
```

for application-specific errors.

---

# 🟢 Subtask 4.2: Comprehensive State Analysis

Install jq if required:

```bash
sudo dnf install jq -y
```

---

## View Complete State Information

```bash
podman inspect my_nginx \
--format '{{json .State}}' | jq
```

---

### Example Output

```json
{
  "Running": true,
  "Paused": false,
  "Restarting": false,
  "ExitCode": 0,
  "StartedAt": "2025-05-30T10:45:00Z"
}
```

---

## State Fields Explained

| Field      | Description         |
| ---------- | ------------------- |
| Running    | Container active    |
| Paused     | Container paused    |
| Restarting | Restart in progress |
| ExitCode   | Last exit code      |
| StartedAt  | Start timestamp     |

---

# 📊 Inspection Workflow

```text
Container/Image
        │
        ▼
 podman inspect
        │
        ▼
 JSON Metadata
        │
 ┌──────┼──────┐
 ▼      ▼      ▼

Env   Ports  Mounts
Vars         

        │
        ▼
 Troubleshooting
```

---

# 🧪 Final Verification

## Verify Container Metadata

```bash
podman inspect my_nginx
```

---

## Verify Environment Variables

```bash
podman inspect env_test \
--format '{{.Config.Env}}'
```

---

## Verify Volume Mounts

```bash
podman inspect vol_test \
--format '{{.Mounts}}'
```

---

## Verify Port Mappings

```bash
podman inspect my_nginx \
--format '{{.NetworkSettings.Ports}}'
```

---

## Verify Exit Codes

```bash
podman inspect fail_test \
--format '{{.State.ExitCode}}'
```

---

# 🔍 Troubleshooting Guide

## Problem 1: Container Not Found

Check available containers:

```bash
podman ps -a
```

---

## Problem 2: Empty IP Address

Verify network:

```bash
podman network ls
```

---

## Problem 3: Missing Port Information

Verify container was started with:

```bash
-p HOST_PORT:CONTAINER_PORT
```

---

## Problem 4: jq Not Installed

Install jq:

```bash
sudo dnf install jq -y
```

or

```bash
sudo apt install jq -y
```

---

# 🚀 Additional Exercises

### Exercise 1

Create a container with multiple environment variables and extract them individually.

Example:

```bash
-e APP_ENV=prod \
-e APP_VERSION=1.0 \
-e APP_REGION=us-east
```

---

### Exercise 2

Compare inspection output between:

✅ Running Container

✅ Stopped Container

Observe changes in:

```text
State
Network
Process Information
```

---

### Exercise 3

Create a script that identifies containers with non-zero exit codes.

Example Logic:

```bash
podman ps -a
podman inspect
check ExitCode
report failures
```

---

# 🧹 Cleanup

Stop containers:

```bash
podman stop my_nginx env_test vol_test
```

---

Remove containers:

```bash
podman rm my_nginx env_test vol_test fail_test
```

---

Remove image:

```bash
podman rmi nginx:alpine
```

---

Verify cleanup:

```bash
podman ps -a
podman images
```

---

# 🎓 Lab Summary

In this lab, you successfully learned how to:

✅ Inspect container metadata

✅ Inspect image metadata

✅ Extract environment variables

✅ Analyze volume mounts

✅ Review network port mappings

✅ Interpret container states

✅ Understand exit codes

✅ Troubleshoot runtime issues using Podman inspection tools

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Inspecting Containers and Images with Podman** lab.

You now possess essential container troubleshooting and inspection skills used daily by:

🚀 DevOps Engineers

🚀 Platform Engineers

🚀 OpenShift Administrators

🚀 Kubernetes Operators

🚀 Site Reliability Engineers (SREs)

These inspection techniques are fundamental for debugging, monitoring, and managing production containerized workloads.

Happy Learning and Happy Containerizing! 🎯
