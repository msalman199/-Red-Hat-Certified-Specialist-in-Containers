# 📜 Retrieving Container Logs and Events with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Logging-Container_Logs-blue?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/Monitoring-Events-green?style=for-the-badge&logo=prometheus" />
  <img src="https://img.shields.io/badge/Troubleshooting-Debugging-orange?style=for-the-badge&logo=gnu-bash" />
  <img src="https://img.shields.io/badge/DevOps-Observability-red?style=for-the-badge&logo=redhat" />
</p>

---

# 📖 Overview

Monitoring container logs and lifecycle events is a critical skill for DevOps Engineers, Platform Engineers, and OpenShift Administrators. Podman provides powerful tools to inspect logs, monitor events, and troubleshoot containerized applications.

In this lab, you will learn how to:

✅ View and analyze container logs

✅ Stream logs in real time

✅ Filter logs by time ranges

✅ Configure alternative log drivers

✅ Monitor container lifecycle events

✅ Troubleshoot failed containers using logs and events

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 View and filter container logs using Podman

🔹 Monitor container lifecycle events

🔹 Configure alternative log drivers

🔹 Analyze logs for troubleshooting

🔹 Filter events for specific containers

🔹 Debug failed container deployments

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement               | Description                   |
| ------------------------- | ----------------------------- |
| 🐳 Podman                 | Version 3.0+ Recommended      |
| 🐧 Linux System           | Any modern Linux distribution |
| 🌐 Internet Access        | Required for pulling images   |
| 💻 Terminal Access        | Bash Shell                    |
| 📚 Basic Podman Knowledge | Familiarity with containers   |

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

## 🔹 Pull Sample Container Image

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

# 📌 Task 1: Viewing Container Logs

---

# 🟢 Subtask 1.1: Basic Log Viewing

## Run Container in Detached Mode

```bash
podman run -d \
--name nginx-container \
docker.io/library/nginx:alpine
```

---

## View Container Logs

```bash
podman logs nginx-container
```

---

### Expected Output

```text
/nginx-entrypoint.sh
Configuration complete
Starting nginx...
```

---

## 💡 What Are Container Logs?

Container logs contain:

✅ Application output

✅ Startup information

✅ Errors and warnings

✅ Debugging information

✅ Runtime events

---

# 🟢 Subtask 1.2: Follow Logs in Real Time

Stream logs continuously:

```bash
podman logs --follow nginx-container
```

or:

```bash
podman logs -f nginx-container
```

---

## Expected Behavior

New log entries appear immediately as they are generated.

---

## Stop Following Logs

Press:

```text
CTRL + C
```

---

## Real-Time Monitoring Flow

```text
Container
    │
    ▼
 Application
    │
    ▼
 Container Logs
    │
    ▼
 podman logs -f
    │
    ▼
 Terminal
```

---

# 🟢 Subtask 1.3: Filter Logs by Time

---

## View Logs from Last 5 Minutes

```bash
podman logs --since 5m nginx-container
```

---

## View Logs from Last Hour

```bash
podman logs --since 1h nginx-container
```

---

## View Logs Between Specific Times

```bash
podman logs \
--since 2023-01-01T00:00:00 \
--until 2023-01-01T12:00:00 \
nginx-container
```

---

## Supported Time Formats

| Example           | Description        |
| ----------------- | ------------------ |
| 5m                | Last 5 minutes     |
| 1h                | Last hour          |
| 24h               | Last day           |
| RFC3339 Timestamp | Specific date/time |

---

# 📌 Task 2: Configuring Alternative Log Drivers

---

# 🟢 Subtask 2.1: Using JSON File Logging

## Start Container with JSON Log Driver

```bash
podman run -d \
--name json-logger \
--log-driver json-file \
docker.io/library/nginx:alpine
```

---

## Verify Log Location

```bash
podman inspect \
--format '{{.HostConfig.LogConfig.Path}}' \
json-logger
```

---

## Expected Output

```text
/path/to/container/logfile.log
```

---

## Benefits of JSON Logging

✅ Easy parsing

✅ Integration with log collectors

✅ Structured log format

✅ Compatible with ELK Stack

---

## JSON Logging Architecture

```text
Container
    │
    ▼
 JSON Logs
    │
    ▼
 Log File
    │
    ▼
 ELK / Splunk
```

---

# 🟢 Subtask 2.2: Using Journald Logging

---

## Run Container with Journald Driver

```bash
podman run -d \
--name journald-logger \
--log-driver journald \
docker.io/library/nginx:alpine
```

---

## View Logs with Journalctl

```bash
journalctl CONTAINER_NAME=journald-logger
```

---

### Alternative Command

```bash
journalctl -xe
```

---

## Benefits of Journald

✅ Centralized logging

✅ Native Linux integration

✅ System-wide search

✅ Better security controls

---

# 📌 Task 3: Monitoring Container Events

---

# 🟢 Subtask 3.1: Basic Event Monitoring

Open Terminal 1:

```bash
podman events --format "{{.Time}} {{.Type}} {{.Status}}"
```

---

Open Terminal 2:

```bash
podman run -d \
--name event-test \
docker.io/library/nginx:alpine
```

---

## Expected Event Output

```text
2025-05-30 container create
2025-05-30 container start
```

---

## What Are Container Events?

Events track:

✅ Container Creation

✅ Container Start

✅ Container Stop

✅ Container Removal

✅ Image Pulls

✅ Network Changes

---

## Event Lifecycle

```text
Create
   │
   ▼
Start
   │
   ▼
Running
   │
   ▼
Stop
   │
   ▼
Remove
```

---

# 🟢 Subtask 3.2: Filtering Events

---

## Filter Start Events

```bash
podman events --filter event=start
```

---

## Filter Specific Container

```bash
podman events --filter container=event-test
```

---

## Filter by Time Range

```bash
podman events \
--filter event=die \
--since 1h
```

---

## Common Event Filters

| Filter         | Purpose            |
| -------------- | ------------------ |
| event=start    | Start events       |
| event=stop     | Stop events        |
| event=die      | Crashes            |
| container=name | Specific container |
| image=name     | Specific image     |

---

# 📌 Task 4: Troubleshooting with Logs and Events

---

# 🟢 Subtask 4.1: Analyze a Failing Container

Run invalid command:

```bash
podman run \
--name failing-container \
docker.io/library/alpine \
/bin/nonexistent-command
```

---

## Expected Failure

```text
Error:
executable file not found
```

---

## View Logs

```bash
podman logs failing-container
```

---

## Check Related Events

```bash
podman events \
--filter container=failing-container \
--since 1m
```

---

### Expected Events

```text
create
start
die
```

---

## Troubleshooting Workflow

```text
Container Failure
        │
        ▼
   podman logs
        │
        ▼
 Identify Error
        │
        ▼
  podman events
        │
        ▼
 Find Failure Cause
```

---

# 🟢 Subtask 4.2: Debugging with Detailed Logs

Run container with debug logging:

```bash
podman run -d \
--name debug-container \
--log-level=debug \
docker.io/library/nginx:alpine
```

---

## View Detailed Logs

```bash
podman logs debug-container
```

---

## Benefits of Debug Logging

✅ More diagnostic information

✅ Startup troubleshooting

✅ Configuration verification

✅ Root cause analysis

---

# 📊 Logging Drivers Comparison

| Feature            | k8s-file | json-file | journald |
| ------------------ | -------- | --------- | -------- |
| Default Podman     | ✅        | ❌         | ❌        |
| Structured Logs    | ❌        | ✅         | ✅        |
| Centralized Search | ❌        | ❌         | ✅        |
| ELK Integration    | ⚠️       | ✅         | ✅        |
| Linux Native       | ❌        | ❌         | ✅        |

---

# 🧪 Final Verification

## Verify Running Containers

```bash
podman ps
```

---

## Verify Logs

```bash
podman logs nginx-container
```

---

## Verify Log Streaming

```bash
podman logs -f nginx-container
```

---

## Verify Events

```bash
podman events --filter container=event-test
```

---

## Verify Journald Logging

```bash
journalctl CONTAINER_NAME=journald-logger
```

---

# 🔍 Troubleshooting Guide

## Problem 1: No Logs Available

Check container state:

```bash
podman ps -a
```

---

## Problem 2: Container Exits Immediately

Inspect logs:

```bash
podman logs <container-name>
```

---

## Problem 3: Missing Events

Verify Podman service:

```bash
podman system info
```

---

## Problem 4: Journald Logs Not Showing

Verify systemd journal service:

```bash
systemctl status systemd-journald
```

---

# 🚀 Real-World DevOps Use Cases

Container logging and event monitoring are essential for:

🔹 Production Troubleshooting

🔹 Incident Response

🔹 Security Auditing

🔹 Application Monitoring

🔹 Kubernetes Debugging

🔹 OpenShift Operations

🔹 CI/CD Pipeline Analysis

🔹 Compliance Reporting

---

# 📚 Additional Resources

### Podman Logs Documentation

```bash
man podman-logs
```

### Podman Events Documentation

```bash
man podman-events
```

### Journalctl Documentation

```bash
man journalctl
```

### Official Podman Documentation

https://docs.podman.io

### Journald Documentation

https://www.freedesktop.org/software/systemd/man/journalctl.html

---

# 🧹 Cleanup

Remove all lab containers:

```bash
podman rm -f \
nginx-container \
json-logger \
journald-logger \
event-test \
failing-container \
debug-container
```

Verify:

```bash
podman ps -a
```

Expected:

```text
No containers found
```

---

# 🎓Summary

In this lab, you successfully learned how to:

✅ View container logs

✅ Follow logs in real time

✅ Filter logs by timestamps

✅ Configure JSON log driver

✅ Configure Journald log driver

✅ Monitor container lifecycle events

✅ Filter and analyze events

✅ Troubleshoot failed containers

✅ Use logs and events for root cause analysis

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Retrieving Container Logs and Events with Podman** lab.

You now possess essential observability and troubleshooting skills used daily by:

🚀 DevOps Engineers

🚀 Platform Engineers

🚀 Site Reliability Engineers (SREs)

🚀 Kubernetes Administrators

🚀 OpenShift Engineers

These skills are fundamental for managing and debugging containerized applications in production environments.

Happy Learning and Happy Containerizing! 🎯
