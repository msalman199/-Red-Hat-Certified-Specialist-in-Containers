# 🔐 Setting WORKDIR and USER Instructions in Containers

<div align="center">

# 🚀 Container Security: WORKDIR & USER

### Learn How to Set Working Directories and Secure Containers with Non-Root Users

![Podman](https://img.shields.io/badge/Podman-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Security](https://img.shields.io/badge/Container_Security-FF6B6B?style=for-the-badge&logo=securityscorecard&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![DevOps](https://img.shields.io/badge/DevOps-0A66C2?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

# 📖 Overview

In containerized environments, security is critical. Two important Dockerfile/Containerfile instructions help improve structure and security:

- 📁 `WORKDIR` → Defines working directory inside container
- 👤 `USER` → Runs container as non-root user

Running containers as root is dangerous because it can allow full system access in case of compromise.

This lab demonstrates how to securely configure containers using best practices.

---

# 🎯 Objectives

By the end of this lab, you will be able to:

- ✅ Set working directory using WORKDIR
- ✅ Create and switch to non-root users
- ✅ Run containers securely as non-root
- ✅ Validate permissions and access restrictions
- ✅ Understand container security principles
- ✅ Apply least privilege model in containers

---

# 📋 Prerequisites

Before starting, ensure you have:

| Requirement | Description |
|------------|------------|
| 🐳 Podman / Docker | Installed and configured |
| 💻 Linux System | Ubuntu, Fedora, CentOS, RHEL |
| 📝 Text Editor | VS Code, Nano, Vim |
| 📘 Basic Knowledge | Containers & Linux commands |

---

# ⚙️ Lab Setup

## 🔍 Step 1: Verify Podman Installation

```bash
podman --version
```

### Expected Output

```text
podman version 4.x.x
```

---

## 📁 Step 2: Create Lab Directory

```bash
mkdir container-security-lab

cd container-security-lab
```

---

# 📁 Task 1: Using WORKDIR in Containerfile

## 🎯 Objective

Understand how WORKDIR sets the default working directory.

---

## 📌 Subtask 1.1: Create Containerfile

```bash
touch Containerfile
```

---

## 📄 Add Base Configuration

```dockerfile
FROM registry.access.redhat.com/ubi9/ubi-minimal

LABEL maintainer="Your Name <your.email@example.com>"
```

---

## 📌 Subtask 1.2: Implement WORKDIR

```dockerfile
WORKDIR /app

RUN pwd > /tmp/workdir.log && whoami >> /tmp/workdir.log
```

---

## 🏗️ Build Image

```bash
podman build -t workdir-demo .
```

---

## ▶️ Run Container

```bash
podman run --rm workdir-demo cat /tmp/workdir.log
```

---

### ✅ Expected Output

```text
/app
root
```

---

## 📚 Key Concept

### WORKDIR

- Sets working directory for all future instructions
- Acts like `cd /app`
- Applies at build time AND runtime

---

# 👤 Task 2: Creating a Non-Root User

## 🎯 Objective

Improve container security by avoiding root execution.

---

## 📌 Subtask 2.1: Create User

Add to Containerfile:

```dockerfile
RUN microdnf install shadow-utils && \
    useradd -u 1001 -d /home/appuser -m appuser && \
    chown -R appuser:appuser /app
```

---

## 📌 Subtask 2.2: Switch User

```dockerfile
USER appuser

RUN whoami >> /tmp/user.log && ls -ld /app >> /tmp/user.log
```

---

## 🏗️ Build Image

```bash
podman build -t nonroot-demo .
```

---

## ▶️ Verify Output

```bash
podman run --rm nonroot-demo cat /tmp/user.log
```

---

### ✅ Expected Output

```text
appuser
drwxr-xr-x 2 appuser appuser /app
```

---

## 🔐 Security Note

Running as non-root:

- ❌ Prevents system-wide changes
- ❌ Limits attack surface
- ✅ Enforces least privilege
- ✅ Improves production security

---

# 🧪 Task 3: Security Validation

## 🚫 Subtask 3.1: Privilege Test

Try restricted operation:

```bash
podman run --rm nonroot-demo touch /sys/kernel/profiling
```

### ❌ Expected Output

```text
Permission denied
```

---

## 🔍 Subtask 3.2: Verify User Context

```bash
podman run -d --name testuser nonroot-demo sleep 300

podman exec testuser ps -ef
```

---

### Expected Result

All processes run as:

```text
appuser
```

---

# ⚖️ Task 4: Security Comparison

## 👑 Run as Root

```bash
podman run --rm --user root workdir-demo whoami
```

Output:

```text
root
```

---

## 👤 Run as Non-Root

```bash
podman run --rm nonroot-demo whoami
```

Output:

```text
appuser
```

---

## 🔥 Key Difference

| Feature | Root User | Non-Root User |
|----------|----------|---------------|
| System Access | Full | Limited |
| Security Risk | High | Low |
| File Access | Unrestricted | Restricted |
| Best Practice | ❌ No | ✅ Yes |

---

# 🔐 Principle of Least Privilege

```text
Give containers ONLY the permissions they need
NOT more
```

---

# 🏗️ Security Architecture Flow

```text
Container Start
      │
      ▼
WORKDIR /app set
      │
      ▼
User Created (appuser)
      │
      ▼
USER switched
      │
      ▼
Limited Permissions Enforced
      │
      ▼
Secure Execution
```

---

# 🚨 Troubleshooting Tips

## ⚠️ Permission Issues

```bash
sudo podman build .
```

---

## ⚠️ User Creation Errors

Ensure package exists:

```bash
microdnf install shadow-utils
```

---

## ⚠️ File Permission Issues

Fix ownership:

```bash
chown -R appuser:appuser /app
```

---

## ⚠️ SELinux Issues

```bash
sudo setenforce 0
```

---

# 📚 Best Practices

| Practice | Recommendation |
|----------|---------------|
| Use WORKDIR | ✅ Always |
| Run as root | ❌ Avoid |
| Use USER directive | ✅ Always |
| Assign least privilege | ✅ Always |
| Fix permissions | ✅ Required |

---

# 🎉 Conclusion

In this lab, you successfully:

- ✅ Configured WORKDIR for structured execution
- ✅ Created non-root users inside containers
- ✅ Switched execution using USER directive
- ✅ Verified security restrictions
- ✅ Applied least privilege principle

---

# 🔐 Key Takeaway

```text
WORKDIR → Organizes container filesystem

USER → Secures container execution
```

---

# 🚀 Next Steps

- 🔹 Learn Podman user namespaces
- 🔹 Use --userns=keep-id
- 🔹 Implement read-only containers
- 🔹 Explore security scanning tools
- 🔹 Study container hardening guides

---

# 🧹 Cleanup

## Remove Images

```bash
podman rmi workdir-demo nonroot-demo
```

---

## Remove Lab Directory

```bash
rm -rf container-security-lab
```

---

# 📚 Additional Resources

- https://docs.podman.io/
- https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
- https://www.redhat.com/en/topics/containers/what-is-container-security

---

<div align="center">

### 🔐 Secure Containers with WORKDIR & USER

**Happy Learning & Secure Building! 🚀**

</div>
