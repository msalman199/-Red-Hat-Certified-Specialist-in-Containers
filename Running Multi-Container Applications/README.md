# 🚀 Running Multi-Container Applications with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Compose-Podman_Compose-blue?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/Kubernetes-Podman_Play_Kube-326CE5?style=for-the-badge&logo=kubernetes" />
  <img src="https://img.shields.io/badge/Database-PostgreSQL-336791?style=for-the-badge&logo=postgresql" />
  <img src="https://img.shields.io/badge/Cache-Redis-DC382D?style=for-the-badge&logo=redis" />
  <img src="https://img.shields.io/badge/Web-Python-3776AB?style=for-the-badge&logo=python" />
</p>

---

# 📖 Overview

Modern applications typically consist of multiple services working together. Instead of running a single container, organizations deploy:

✅ Web Applications

✅ Databases

✅ Cache Services

✅ Message Queues

✅ Background Workers

Podman provides powerful tools for managing these applications through:

🔹 **podman-compose**

🔹 **Podman Pods**

🔹 **podman play kube**

🔹 **Kubernetes YAML Deployments**

In this lab, you will build and deploy a complete multi-container application stack using Podman.

---

# 🎯 Objectives

By the end of this lab, you will be able to:

✅ Define multi-container applications using Podman Compose

✅ Configure persistent storage using volumes

✅ Create container networks

✅ Manage application secrets

✅ Deploy applications with podman-compose

✅ Generate Kubernetes YAML files

✅ Deploy Kubernetes manifests using podman play kube

✅ Understand production-ready container orchestration concepts

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement | Description |
|------------|-------------|
| 🐳 Podman | Version 3.0+ |
| 🔧 Podman Compose | Installed |
| 🐧 Linux System | RHEL, Rocky, AlmaLinux, Fedora, Ubuntu |
| 📚 YAML Knowledge | Basic Understanding |
| 🌐 Internet Access | To pull container images |
| 💻 Terminal Access | Bash Shell |

---

# 🔧 Technologies Used

| Technology | Purpose |
|------------|----------|
| 🐳 Podman | Container Runtime |
| 📦 Podman Compose | Multi-Container Management |
| ☸️ Kubernetes YAML | Container Deployment |
| 🐍 Python | Web Service |
| 🗄️ PostgreSQL | Database |
| ⚡ Redis | In-Memory Cache |
| 🔐 Podman Secrets | Secure Credentials |
| 🌉 Bridge Network | Container Communication |

---

# 🏗️ Lab Setup

---

## 🔹 Start Podman Service

```bash
systemctl start podman
```

---

## 🔹 Verify Installation

```bash
podman --version
podman-compose --version
```

### Expected Output

```bash
podman version 4.x.x
podman-compose version 1.x.x
```

---

# 📌 Task 1: Create and Deploy a Multi-Container Application

---

# 🟢 Subtask 1.1: Create Project Directory

```bash
mkdir multi-container-lab
cd multi-container-lab
```

---

# 🟢 Subtask 1.2: Create docker-compose.yml

Create the following file:

```yaml
version: '3.8'

services:

  web:
    image: docker.io/python:3.9
    command: python -m http.server 8000

    ports:
      - "8000:8000"

    volumes:
      - ./app:/app

    depends_on:
      - redis
      - db

    networks:
      - app-network

  redis:
    image: docker.io/redis:alpine

    ports:
      - "6379:6379"

    volumes:
      - redis-data:/data

    networks:
      - app-network

  db:
    image: docker.io/postgres:13-alpine

    environment:
      POSTGRES_PASSWORD: example

    volumes:
      - postgres-data:/var/lib/postgresql/data

    networks:
      - app-network

volumes:
  redis-data:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

---

## 📊 Architecture Overview

```text
                    🌐 Browser
                         │
                         ▼

                 localhost:8000
                         │
                         ▼

               ┌─────────────────┐
               │  Python Web App │
               └────────┬────────┘
                        │

          ┌─────────────┴─────────────┐
          ▼                           ▼

   ┌─────────────┐           ┌─────────────┐
   │    Redis    │           │ PostgreSQL │
   │    Cache    │           │  Database  │
   └─────────────┘           └─────────────┘

          ▲                           ▲
          │                           │

      Redis Volume             PostgreSQL Volume
```

---

## 🔍 Understanding the Compose File

### 🐍 Web Service

Provides a simple Python HTTP server.

```yaml
web:
```

---

### ⚡ Redis Service

Acts as a caching layer.

```yaml
redis:
```

---

### 🗄️ PostgreSQL Service

Stores persistent application data.

```yaml
db:
```

---

### 💾 Volumes

Ensure data survives container restarts.

```yaml
volumes:
```

---

### 🌉 Networks

Allow containers to communicate securely.

```yaml
networks:
```

---

# 📌 Task 2: Deploy the Application Stack

---

# 🟢 Subtask 2.1: Start Services

```bash
podman-compose up -d
```

---

# 🟢 Subtask 2.2: Verify Running Containers

```bash
podman ps
```

Expected Output:

```text
web
redis
db
```

---

# 🟢 Subtask 2.3: Test Web Service

```bash
curl http://localhost:8000
```

Expected Output:

```html
Directory listing for /
```

or

```text
Python HTTP Server Response
```

---

# 🔍 Verify Container Network

```bash
podman network ls
```

Expected:

```text
app-network
```

---

# 📌 Task 3: Add Secrets Management

---

# 🟢 Subtask 3.1: Create Database Secret

```bash
echo "supersecret" | podman secret create db_password -
```

---

## Verify Secret

```bash
podman secret ls
```

Expected:

```text
db_password
```

---

# 🟢 Subtask 3.2: Update Compose File

Replace the database section with:

```yaml
db:
  image: docker.io/postgres:13-alpine

  secrets:
    - db_password

  environment:
    POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    external: true
```

---

# 🔍 Secret Flow

```text
Secret Value
     │
     ▼

Podman Secret Store
     │
     ▼

/run/secrets/db_password
     │
     ▼

PostgreSQL Container
```

---

# 🟢 Subtask 3.3: Redeploy Application

Stop services:

```bash
podman-compose down
```

Start again:

```bash
podman-compose up -d
```

---

## Troubleshooting SELinux Issues

If permission errors occur:

```bash
sudo setsebool -P container_manage_cgroup true
```

---

# 📌 Task 4: Deploy Using Kubernetes YAML

---

# 🟢 Subtask 4.1: Generate Kubernetes Manifest

Generate deployment YAML:

```bash
podman kube generate \
--service \
-f k8s-deployment.yaml \
web redis db
```

---

## Examine Generated YAML

```bash
cat k8s-deployment.yaml
```

Expected Sections:

```yaml
apiVersion:
kind:
metadata:
spec:
containers:
```

---

# 🟢 Subtask 4.2: Deploy with Podman Play Kube

Deploy application:

```bash
podman play kube k8s-deployment.yaml
```

---

## Verify Pod Creation

```bash
podman pod ps
```

Expected:

```text
POD ID
NAME
STATUS
```

---

## Verify Containers

```bash
podman ps
```

Expected:

```text
web
redis
db
```

---

# 📊 Podman Compose vs Play Kube

| Feature | Podman Compose | Play Kube |
|----------|---------------|-----------|
| Easy Setup | ✅ | ⚠️ |
| YAML Format | Compose | Kubernetes |
| Kubernetes Compatible | ❌ | ✅ |
| Learning Curve | Easy | Medium |
| Production Ready | ⚠️ | ✅ |
| OpenShift Alignment | ⚠️ | ✅ |

---

# 🔍 Useful Verification Commands

---

## View Running Containers

```bash
podman ps
```

---

## View Pods

```bash
podman pod ps
```

---

## View Networks

```bash
podman network ls
```

---

## View Volumes

```bash
podman volume ls
```

---

## View Secrets

```bash
podman secret ls
```

---

## View Logs

```bash
podman logs <container_name>
```

Example:

```bash
podman logs web
```

---

# 🚨 Troubleshooting Guide

---

## Problem 1: Containers Not Starting

Check logs:

```bash
podman logs web
podman logs redis
podman logs db
```

---

## Problem 2: Port Already In Use

Check:

```bash
ss -tulnp | grep 8000
```

Change port mapping if necessary.

---

## Problem 3: Secret Not Found

Verify:

```bash
podman secret ls
```

Recreate:

```bash
echo "supersecret" | podman secret create db_password -
```

---

## Problem 4: Network Communication Issues

Inspect network:

```bash
podman network inspect app-network
```

---

## Problem 5: Volume Problems

Check volumes:

```bash
podman volume ls
```

---

# 🎯 Real-World DevOps Applications

These techniques are used daily for:

✅ Microservices Architectures

✅ OpenShift Deployments

✅ Kubernetes Applications

✅ CI/CD Pipelines

✅ Stateful Applications

✅ Multi-Tier Web Applications

✅ Cloud-Native Platforms

✅ Enterprise Container Environments

---

# 🧪 Final Verification

Verify all components:

```bash
podman ps
podman pod ps
podman volume ls
podman secret ls
```

Test application:

```bash
curl http://localhost:8000
```

Expected:

```text
Successful Response
```

---

# 🧹 Cleanup

---

## Stop Compose Application

```bash
podman-compose down
```

---

## Remove Pods

```bash
podman pod rm -a -f
```

---

## Remove Containers

```bash
podman rm -a -f
```

---

## Remove Volumes

```bash
podman volume prune
```

---

## Remove Secret

```bash
podman secret rm db_password
```

---

## Verify Cleanup

```bash
podman ps -a
podman pod ps
podman volume ls
```

---

# 📚 Additional Resources

### 📖 Podman Documentation

https://podman.io/docs

---

### 📖 Compose Specification

https://compose-spec.io

---

### 📖 Kubernetes Documentation

https://kubernetes.io/docs

---

### 📖 Podman Play Kube Guide

https://docs.podman.io/en/latest/markdown/podman-play-kube.1.html

---

# 🎓 Lab Summary

In this lab, you successfully learned how to:

✅ Create multi-container applications

✅ Deploy services with Podman Compose

✅ Configure networks and volumes

✅ Manage application secrets

✅ Generate Kubernetes YAML files

✅ Deploy applications using podman play kube

✅ Work with production-style container architectures

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Running Multi-Container Applications with Podman** lab.

You now possess essential skills used by:

🚀 DevOps Engineers

🚀 Cloud Engineers

🚀 Platform Engineers

🚀 OpenShift Administrators

🚀 Kubernetes Administrators

🚀 Site Reliability Engineers (SREs)

These skills are foundational for managing modern cloud-native and containerized applications in enterprise environments.

Happy Learning & Happy Containerizing! 🚀🐳☸️
