# 🚀 EXPOSE and Port Binding with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Python-3.9-blue?style=for-the-badge&logo=python" />
  <img src="https://img.shields.io/badge/Flask-Web_App-black?style=for-the-badge&logo=flask" />
  <img src="https://img.shields.io/badge/Linux-Networking-orange?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/Containerfile-EXPOSE-green?style=for-the-badge&logo=docker" />
</p>

---

# 📖 Overview

This lab demonstrates how to expose ports within a container image and publish those ports to the host system using Podman.

You will learn:

✅ How the `EXPOSE` instruction works

✅ The difference between exposing and publishing ports

✅ Port mapping between host and container

✅ Testing network connectivity

✅ Troubleshooting common port conflicts

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 Understand how to document and expose ports in container images

🔹 Learn to map container ports to host ports

🔹 Test network connectivity between containers and host systems

🔹 Troubleshoot common port binding issues

---

# 🛠️ Prerequisites

Ensure the following tools are installed:

| Requirement | Description |
|------------|-------------|
| 🐳 Podman | Version 3.0+ recommended |
| 🐧 Linux CLI | Basic command-line knowledge |
| 📝 Text Editor | vim, nano, gedit, VS Code |
| 🌐 Network Access | Required for image downloads |
| 🔍 curl / telnet | Connectivity testing |

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

## 🔹 Create Working Directory

```bash
mkdir portbinding-lab
cd portbinding-lab
```

---

# 📌 Task 1: Using EXPOSE in Containerfile

---

## 🟢 Subtask 1.1: Create a Simple Web Application

### Create app.py

```bash
cat > app.py <<EOF
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from the exposed container port!\n"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
EOF
```

### Create requirements.txt

```bash
echo "flask" > requirements.txt
```

---

## 🟢 Subtask 1.2: Create Containerfile with EXPOSE

Create a Containerfile:

```bash
cat > Containerfile <<EOF
FROM python:3.9-slim

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

EXPOSE 8080

CMD ["python", "app.py"]
EOF
```

---

## 💡 Key Concept

### What Does EXPOSE Do?

```dockerfile
EXPOSE 8080
```

✔ Documents the port used by the application

✔ Provides metadata for container users

✔ Does NOT publish the port automatically

❌ Does NOT make the port accessible from the host

---

# 📌 Task 2: Run Container with Port Mappings

---

## 🟢 Subtask 2.1: Build the Image

Build the container image:

```bash
podman build -t exposed-app .
```

---

## 🟢 Subtask 2.2: Run with Port Mapping

Map:

```text
Host Port      →      Container Port
8080           →      8080
```

Run the container:

```bash
podman run -d -p 8080:8080 --name webapp exposed-app
```

---

### Verify Running Container

```bash
podman ps
```

Expected output should show:

```text
0.0.0.0:8080->8080/tcp
```

---

## ⚠️ Troubleshooting Tip

If port 8080 is already occupied:

```bash
podman run -d -p 8081:8080 --name webapp exposed-app
```

This maps:

```text
Host 8081 → Container 8080
```

---

# 📌 Task 3: Test Connectivity from Host

---

## 🟢 Subtask 3.1: Verify Port Mapping

Check listening ports:

```bash
ss -tulnp | grep 8080
```

---

### Test the Application

```bash
curl http://localhost:8080
```

Expected output:

```text
Hello from the exposed container port!
```

---

## 🟢 Subtask 3.2: Test Different Port Mapping

### Stop Existing Container

```bash
podman stop webapp
```

---

### Start Using Host Port 9090

```bash
podman run -d -p 9090:8080 --name webapp2 exposed-app
```

---

### Test New Port

```bash
curl http://localhost:9090
```

Expected output:

```text
Hello from the exposed container port!
```

---

# 📌 Task 4: Troubleshoot Port Conflicts

---

## 🟢 Subtask 4.1: Simulate Port Conflict

Attempt:

```bash
podman run -d -p 8080:8080 --name webapp3 exposed-app
```

Expected error:

```text
Error: port already in use
```

---

## 🟢 Subtask 4.2: Resolve Conflict

### Identify Conflicting Process

```bash
sudo lsof -i :8080
```

Example output:

```text
COMMAND   PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
python3  2456 root  3u IPv4 12345 0t0 TCP *:8080 (LISTEN)
```

---

## ✅ Resolution Options

### Option 1: Stop the Conflicting Service

```bash
sudo kill <PID>
```

---

### Option 2: Use a Different Host Port

```bash
podman run -d -p 8081:8080 exposed-app
```

---

### Option 3: Let Podman Select a Port

```bash
podman run -d -P --name webapp4 exposed-app
```

View assigned port:

```bash
podman port webapp4
```

Example:

```text
8080/tcp -> 0.0.0.0:32768
```

---

# 🔍 Understanding Port Mapping

## 📊 Port Mapping Diagram

```text
┌─────────────────────────┐
│      Host Machine       │
│                         │
│  localhost:8080         │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│       Container         │
│                         │
│ Application :8080       │
└─────────────────────────┘
```

---

# 🧪 Final Verification

---

## Verify All Containers

```bash
podman ps -a
```

---

## Verify Port Mappings

```bash
podman port webapp2
```

---

## Test Connectivity

```bash
curl http://localhost:9090
```

Expected:

```text
Hello from the exposed container port!
```

---

# 🧹 Cleanup

Stop all containers:

```bash
podman stop -a
```

Remove all containers:

```bash
podman rm -a
```

Verify cleanup:

```bash
podman ps -a
```

---

# 📚 Additional Resources

### 📖 Podman Documentation

https://docs.podman.io

---

### 📖 Dockerfile EXPOSE Reference

https://docs.docker.com/engine/reference/builder/#expose

---

### 📖 Linux Network Troubleshooting

```bash
man ss
```

```bash
man lsof
```

---

# 🎓 Summary

In this lab you successfully learned:

✅ Using the `EXPOSE` instruction

✅ Building container images with Podman

✅ Publishing container ports to the host

✅ Testing connectivity using curl

✅ Managing custom port mappings

✅ Troubleshooting port conflicts

✅ Inspecting network bindings

✅ Cleaning up container resources

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **EXPOSE and Port Binding with Podman** lab and gained hands-on experience with container networking fundamentals used in modern DevOps, Cloud, Kubernetes, and OpenShift environments.

🚀 Happy Learning & Containerizing!
