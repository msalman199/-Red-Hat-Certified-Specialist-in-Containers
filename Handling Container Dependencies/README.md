# 🔗 Handling Container Dependencies with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Podman-Container_Runtime-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Compose-Multi_Service-blue?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/Health_Checks-Ready-green?style=for-the-badge&logo=heartbeat" />
  <img src="https://img.shields.io/badge/DevOps-Reliability-orange?style=for-the-badge&logo=redhat" />
</p>

---

# 📖 Overview

In real-world containerized systems, applications rarely run as a single service. Instead, they depend on:

- 🗄️ Databases  
- 🌐 Web servers  
- ⚡ Cache systems  
- 🔌 APIs and backend services  

This lab teaches how to manage **service dependencies, startup order, health checks, and retry logic** using Podman Compose.

---

# 🎯 Objectives

By the end of this lab, you will be able to:

✅ Implement `depends_on` in Compose files  
✅ Write and use health check scripts  
✅ Implement retry logic in entrypoint scripts  
✅ Ensure service startup ordering  
✅ Test connectivity between dependent containers  

---

# 🛠️ Prerequisites

| Requirement | Description |
|------------|-------------|
| 🐳 Podman | Installed (recommended for OpenShift) |
| 📦 Podman Compose | Installed |
| 🐧 Linux System | Any modern distribution |
| 💻 Terminal | Bash shell access |
| ✏️ Editor | Vim, Nano, VS Code |

---

# 🏗️ Setup

## 🔹 Create Working Directory

```bash
mkdir container-dependencies-lab
cd container-dependencies-lab
```

---

## 🔹 Verify Installation

```bash
podman --version
podman-compose --version
```

---

# 📌 Task 1: Using `depends_on` in Compose Files

---

## 🟢 Subtask 1.1: Create docker-compose.yml

```yaml
version: '3.8'

services:

  db:
    image: postgres:15-alpine

    environment:
      POSTGRES_PASSWORD: example

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  web:
    image: nginx:alpine

    ports:
      - "8080:80"

    depends_on:
      db:
        condition: service_healthy
```

---

## 🧠 Key Concepts

### 🔹 depends_on
Controls service startup order.

### 🔹 healthcheck
Ensures service is **ready**, not just running.

### 🔹 service_healthy
Waits until dependency passes health checks.

---

## 🚀 Subtask 1.3: Start Services

```bash
podman-compose up -d
```

---

## 📊 Expected Behavior

- Database starts first 🗄️  
- Health check runs ✅  
- Web service starts only after DB is healthy 🌐  

---

# 📌 Task 2: Writing Health Check Scripts

---

## 🟢 Subtask 2.1: Create Health Check Script

Create `healthcheck.sh`:

```sh
#!/bin/sh

# Check database connectivity
if curl -s http://db:5432 | grep -q 'PostgreSQL'; then
  exit 0
else
  exit 1
fi
```

---

## 🔧 Make Executable

```bash
chmod +x healthcheck.sh
```

---

## 🟢 Subtask 2.2: Add to Compose File

```yaml
app:
  image: python:3.9-alpine

  volumes:
    - ./healthcheck.sh:/healthcheck.sh

  healthcheck:
    test: ["CMD", "/healthcheck.sh"]
    interval: 10s
    timeout: 5s
    retries: 3
```

---

## 🧠 Why Health Checks Matter

✔ Prevents traffic to unhealthy services  
✔ Ensures dependency readiness  
✔ Improves system reliability  
✔ Critical for production workloads  

---

# 📌 Task 3: Retry Logic in Entrypoint

---

## 🟢 Subtask 3.1: Create entrypoint.sh

```sh
#!/bin/sh

max_retries=5
retry_delay=5

for i in $(seq 1 $max_retries); do
  if nc -z db 5432; then
    echo "Database is ready!"
    exec "$@"
    break
  else
    echo "Waiting for database... Attempt $i/$max_retries"
    sleep $retry_delay
  fi
done

echo "Failed to connect after $max_retries attempts"
exit 1
```

---

## 🔧 Make Executable

```bash
chmod +x entrypoint.sh
```

---

## 🟢 Subtask 3.2: Create Dockerfile

```Dockerfile
FROM python:3.9-alpine

WORKDIR /app

COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

ENTRYPOINT ["./entrypoint.sh"]
CMD ["python", "app.py"]
```

---

## 🧠 Retry Logic Flow

```text
Start Container
      │
      ▼
Check DB Connection
      │
  ┌─── Yes ───► Run App
  │
  ▼
No (Retry Loop)
      │
Retry until max_retries
      │
Failure Exit
```

---

# 📌 Task 4: Testing Service Connectivity

---

## 🟢 Subtask 4.1: Create Application

```python
import os
import psycopg2

def connect_db():
    try:
        conn = psycopg2.connect(
            host="db",
            database="postgres",
            user="postgres",
            password=os.getenv("POSTGRES_PASSWORD")
        )
        print("Successfully connected to database!")
        conn.close()
        return True
    except Exception as e:
        print(f"Connection failed: {e}")
        return False

if __name__ == "__main__":
    connect_db()
```

---

## 🟢 Subtask 4.2: Update Compose File

```yaml
app:
  build: .

  depends_on:
    db:
      condition: service_healthy

  environment:
    POSTGRES_PASSWORD: example
```

---

## 🚀 Subtask 4.3: Run System

```bash
podman-compose up --build
```

---

## 📊 Expected Outcome

- DB starts first 🗄️  
- Health check validates readiness ✅  
- App waits for DB 🔄  
- App connects successfully 🎯  

---

# 🔍 Troubleshooting Guide

---

## ❌ Containers Not Starting

```bash
podman-compose logs
```

---

## ❌ Health Check Failing

```bash
podman inspect --format='{{.State.Health.Status}}' <container>
```

Fix:

- Increase retries
- Increase interval
- Test script manually

---

## ❌ DB Connection Issues

Check:

```bash
podman network ls
podman inspect <container>
```

---

## ❌ Service Name Mismatch

Ensure:

```yaml
host: db
```

matches service name exactly.

---

# 🧠 Key Concepts Summary

### 🔹 depends_on
Controls startup order (not readiness alone)

### 🔹 healthcheck
Ensures service is fully ready

### 🔹 retry logic
Handles transient failures

### 🔹 service communication
Uses internal DNS (service names)

---

# 📊 Real-World Use Cases

These patterns are used in:

- 🏦 Banking systems
- 🛒 E-commerce platforms
- ☁️ Cloud-native microservices
- 🚀 CI/CD pipelines
- 🧠 AI backend systems
- 📡 API gateway architectures

---

# 🧪 Final Verification

```bash
podman ps
podman-compose logs
```

Test DB connection inside app logs.

---

# 🧹 Cleanup

```bash
podman-compose down
```

```bash
podman system prune -f
```

---

# 🎓 Lab Summary

In this lab, you learned how to:

✅ Manage container dependencies using `depends_on`  
✅ Implement health checks for service readiness  
✅ Build retry logic for reliability  
✅ Test inter-container communication  
✅ Build production-style startup flows  

---

# 🏆 Completion Badge

🎉 Congratulations!

You have successfully completed:

**Handling Container Dependencies with Podman**

You now understand how production systems ensure:

🚀 Reliable startup order  
🚀 Service health validation  
🚀 Fault-tolerant initialization  
🚀 Resilient container communication  

These skills are essential for:

👨‍💻 DevOps Engineers  
☁️ Cloud Engineers  
⚙️ SRE Engineers  
🐳 Kubernetes Developers  
