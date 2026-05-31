# 🔐 Case Study: Build & Deploy a Secure Flask Microservice with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Flask-Microservice-green?style=for-the-badge&logo=flask" />
  <img src="https://img.shields.io/badge/PostgreSQL-Database-blue?style=for-the-badge&logo=postgresql" />
  <img src="https://img.shields.io/badge/Podman-Container_Runtime-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Security-Best_Practices-red?style=for-the-badge&logo=securityscorecard" />
  <img src="https://img.shields.io/badge/Compose-Multi_Container-orange?style=for-the-badge&logo=docker" />
</p>

---

# 📖 Overview

In this case study, you will build a **secure, production-ready Flask microservice** backed by a **PostgreSQL database**, fully containerized using **Podman** and deployed using **Podman Compose**.

You will also apply:

🔐 Security best practices  
📦 Image scanning & signing  
🧩 Multi-container orchestration  
📊 Logging & recovery testing  
🗄️ Persistent storage  

---

# 🎯 Objectives

By the end of this lab, you will be able to:

✅ Containerize a Flask application using Podman  
✅ Deploy PostgreSQL with persistent storage  
✅ Use Podman Compose for multi-container apps  
✅ Implement secrets management securely  
✅ Scan and sign container images  
✅ Test recovery and logging mechanisms  
✅ Build production-ready microservice architecture  

---

# 🛠️ Prerequisites

| Requirement | Description |
|------------|-------------|
| 🐳 Podman | Installed (`dnf install podman podman-compose`) |
| 🐍 Python | Basic knowledge |
| 🐘 PostgreSQL | Basic understanding |
| 💻 Linux System | RHEL / Fedora / Ubuntu |
| 🌐 Internet | Pull images |
| ✏️ Editor | VS Code / Vim / Nano |

---

# 🏗️ Lab Setup

## 🔹 Create Working Directory

```bash
mkdir flask-microservice-lab
cd flask-microservice-lab
```

---

## 🔹 Verify Installation

```bash
podman --version
podman-compose --version
```

---

# 📌 Task 1: Create Flask Application

---

## 🟢 Subtask 1.1: Python Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install flask psycopg2-binary
```

---

## 🟢 Subtask 1.2: Create Flask App

```python
from flask import Flask, jsonify
import psycopg2
import os

app = Flask(__name__)

DB_HOST = os.getenv('DB_HOST', 'localhost')
DB_NAME = os.getenv('DB_NAME', 'mydb')
DB_USER = os.getenv('DB_USER', 'postgres')
DB_PASS = os.getenv('DB_PASS', 'password')

def get_db_connection():
    return psycopg2.connect(
        host=DB_HOST,
        database=DB_NAME,
        user=DB_USER,
        password=DB_PASS
    )

@app.route('/')
def hello():
    return jsonify({"status": "OK", "message": "Flask Microservice Running"})

@app.route('/data')
def get_data():
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute('SELECT version();')
    db_version = cur.fetchone()
    cur.close()
    conn.close()
    return jsonify({"database": db_version[0]})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## 🟢 Subtask 1.3: Requirements File

```bash
pip freeze > requirements.txt
```

---

# 📌 Task 2: Containerize Flask App

---

## 🟢 Subtask 2.1: Create Containerfile

```Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV FLASK_APP=app.py

EXPOSE 5000

CMD ["flask", "run", "--host=0.0.0.0"]
```

---

## 🟢 Subtask 2.2: Build Image

```bash
podman build -t flask-app .
```

---

# 📌 Task 3: Configure PostgreSQL Container

---

## 🟢 Subtask 3.1: Database Init Script

```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);
```

---

## 🟢 Subtask 3.2: Create Secret

```bash
echo "mysecretpassword" > db_password.txt
podman secret create db_password db_password.txt
```

---

# 📌 Task 4: Deploy with Podman Compose

---

## 🟢 Subtask 4.1: podman-compose.yml

```yaml
version: '3'

services:

  web:
    image: flask-app
    build: .

    ports:
      - "5000:5000"

    environment:
      - DB_HOST=db
      - DB_NAME=mydb
      - DB_USER=postgres
      - DB_PASS_FILE=/run/secrets/db_password

    secrets:
      - db_password

    depends_on:
      - db

    networks:
      - app-network

  db:
    image: postgres:13

    volumes:
      - pgdata:/var/lib/postgresql/data

    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
      - POSTGRES_DB=mydb

    secrets:
      - db_password

    ports:
      - "5432:5432"

    networks:
      - app-network

volumes:
  pgdata:

secrets:
  db_password:
    file: db_password.txt

networks:
  app-network:
    driver: bridge
```

---

## 🧠 Architecture Overview

```text
        🌐 Client
           │
           ▼
   http://localhost:5000
           │
   ┌───────┴────────┐
   │   Flask App    │
   │ (web service)  │
   └───────┬────────┘
           │
           ▼
   ┌───────────────┐
   │ PostgreSQL DB  │
   └───────────────┘
           │
           ▼
     Persistent Volume (pgdata)
```

---

## 🟢 Subtask 4.2: Deploy Stack

```bash
podman-compose up -d
```

---

# 📌 Task 5: Security Practices

---

## 🟢 Subtask 5.1: Image Scanning

```bash
podman scan flask-app
podman scan postgres:13
```

---

## 🟢 Subtask 5.2: Image Signing

```bash
podman trust set -t reject default
podman trust set -f ~/.ssh/id_rsa.pub docker.io
```

Tag and push:

```bash
podman tag flask-app localhost/flask-app:latest
podman push localhost/flask-app:latest
```

---

# 📌 Task 6: Testing & Validation

---

## 🟢 Subtask 6.1: Test API

```bash
curl http://localhost:5000
curl http://localhost:5000/data
```

---

## 🟢 Subtask 6.2: View Logs

```bash
podman logs -f flask-microservice-lab_web_1
```

---

## 🟢 Subtask 6.3: Test Recovery

```bash
podman-compose stop db
podman-compose start db
```

Test again:

```bash
curl http://localhost:5000/data
```

---

# 🔍 Troubleshooting Guide

---

## ❌ Container Not Starting

```bash
podman logs <container_name>
```

---

## ❌ DB Connection Issues

✔ Check environment variables  
✔ Verify service name (`db`)  
✔ Ensure secrets are mounted  

---

## ❌ Image Scan Failure

✔ Update Podman  
✔ Check internet connectivity  

---

## ❌ Volume Permission Issues

Add SELinux label:

```bash
:Z
```

---

# 🧠 Key Concepts Summary

### 🔹 Microservice Architecture
Flask service + PostgreSQL backend

### 🔹 Secrets Management
Secure DB credentials using Podman secrets

### 🔹 Persistent Storage
PostgreSQL data survives restarts

### 🔹 Multi-Container Deployment
Handled via Podman Compose

### 🔹 Security Practices
Image scanning + signing

---

# 📊 Real-World Use Cases

This architecture is used in:

- 🏦 Banking systems
- 🛒 E-commerce platforms
- 📡 API backend services
- ☁️ Cloud-native microservices
- 🚀 CI/CD deployments
- 🔐 Secure enterprise applications

---

# 🧪 Final Verification

```bash
podman ps
podman-compose logs
curl http://localhost:5000
```

---

# 🧹 Cleanup

```bash
podman-compose down
```

```bash
podman secret rm db_password
podman rmi flask-app
```

---

# 🎓 Lab Summary

In this case study, you learned how to:

✅ Build a Flask microservice  
✅ Connect PostgreSQL with persistence  
✅ Deploy multi-container apps  
✅ Secure secrets using Podman  
✅ Scan and sign container images  
✅ Test recovery and resilience  

---

# 🏆 Completion Badge

🎉 Congratulations!

You have completed the:

**Secure Flask Microservice Deployment with Podman**

You are now ready for:

🚀 Production container deployments  
🚀 Kubernetes/OpenShift workloads  
🚀 Cloud-native microservices architecture  
🚀 DevOps engineering workflows  

---

# 🚀 Next Steps

- Add Redis caching layer  
- Implement health checks  
- Add CI/CD pipeline integration  
- Deploy to Kubernetes/OpenShift  
```
