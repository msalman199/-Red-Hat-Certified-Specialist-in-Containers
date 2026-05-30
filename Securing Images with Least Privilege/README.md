# 🔐 Securing Images with Least Privilege 

<div align="center">

# 🚀 Container Image Security & Hardening

### Learn Vulnerability Scanning, Minimal Images, Non-Root Users, and Image Signing

![Podman](https://img.shields.io/badge/Podman-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Security](https://img.shields.io/badge/Container_Security-FF6B6B?style=for-the-badge&logo=securityscorecard&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![DevOps](https://img.shields.io/badge/DevOps-0A66C2?style=for-the-badge&logo=githubactions&logoColor=white)
![Podman Scan](https://img.shields.io/badge/Image_Scanning-4CAF50?style=for-the-badge&logo=verizon&logoColor=white)

</div>

---

# 📖 Overview

Container security is based on one core idea:

> 🔐 **Principle of Least Privilege**

This means containers should only have the minimum permissions, packages, and access required to run.

In this lab, you will learn how to:

- Scan images for vulnerabilities
- Reduce attack surface using cleanup
- Use minimal base images
- Run containers as non-root users
- Sign images for provenance verification

---

# 🎯 Objectives

By the end of this lab, you will be able to:

- ✅ Scan container images for vulnerabilities
- ✅ Interpret CVE reports
- ✅ Reduce image size using cache cleanup
- ✅ Use minimal base images (Alpine)
- ✅ Run containers as non-root users
- ✅ Implement image signing (provenance)
- ✅ Apply least privilege security model

---

# 📋 Prerequisites

| Requirement | Description |
|------------|------------|
| 🐳 Podman v3+ | Installed and configured |
| 💻 Linux OS | Ubuntu / Fedora / RHEL |
| 🌐 Internet | Required for image pulling |
| 🔐 sudo access | For system operations |
| 📘 Basic knowledge | Containers & CLI usage |

---

# ⚙️ Lab Setup

## 🔍 Step 1: Verify Podman

```bash
podman --version
```

### Expected Output

```text
podman version 3.x.x
```

---

## 📁 Step 2: Create Working Directory

```bash
mkdir secure_lab

cd secure_lab
```

---

# 🛡️ Task 1: Scan Images for Vulnerabilities

## 🎯 Objective

Identify security issues in container images.

---

## 📌 Subtask 1.1: Pull Image

```bash
podman pull docker.io/library/nginx:latest
```

---

## 📌 Subtask 1.2: Scan Image

```bash
podman scan nginx:latest
```

---

## 📊 Expected Output

- CVE list
- Severity levels
- Package vulnerabilities

---

## 🔍 Subtask 1.3: Interpret Results

Focus on:

| Severity | Meaning |
|----------|--------|
| 🔴 Critical | CVSS ≥ 9 |
| 🟠 High | 7.0 - 8.9 |
| 🟡 Medium | 4.0 - 6.9 |
| 🟢 Low | < 4.0 |

---

# 🧹 Task 2: Remove Package Cache

## 🎯 Objective

Reduce image size and attack surface.

---

## 📌 Subtask 2.1: Create Dockerfile

```dockerfile
FROM docker.io/library/nginx:latest

RUN rm -rf /var/cache/apt/* /var/lib/apt/lists/*
```

---

## 🏗️ Subtask 2.2: Build Image

```bash
podman build -t nginx_clean .
```

---

## 📦 Subtask 2.3: Verify Size

```bash
podman images
```

---

## 🔥 Key Concept

Removing cache:

- Reduces image size
- Removes unnecessary files
- Reduces attack surface

---

# 🪶 Task 3: Use Minimal Base Images

## 🎯 Objective

Use lightweight images to reduce vulnerabilities.

---

## 📌 Subtask 3.1: Alpine-Based Image

```dockerfile
FROM docker.io/library/alpine:latest

RUN apk add --no-cache nginx && \
    rm -rf /var/cache/apk/*
```

---

## 🏗️ Build Image

```bash
podman build -t nginx_alpine .
```

---

## 📊 Compare Sizes

```bash
podman images | grep nginx
```

---

## 🔥 Key Insight

| Base Image | Size | Security |
|------------|------|----------|
| Debian/Ubuntu | Large | More CVEs |
| Alpine | Small | Fewer CVEs |

---

# 👤 Task 4: Run as Non-Root User

## 🎯 Objective

Enforce least privilege execution.

---

## 📌 Subtask 4.1: Create User

```dockerfile
FROM docker.io/library/alpine:latest

RUN apk add --no-cache nginx && \
    adduser -D nginxuser && \
    chown -R nginxuser:nginxuser /var/lib/nginx

USER nginxuser
```

---

## 🏗️ Build Image

```bash
podman build -t nginx_nonroot .
```

---

## ▶️ Run Container

```bash
podman run -d --name secure_nginx -p 8080:80 nginx_nonroot
```

---

## 🔍 Check Processes

```bash
podman exec secure_nginx ps aux
```

---

### Expected Output

```text
nginxuser   nginx process
```

---

## 🔐 Security Benefit

- ❌ No root access
- ❌ No system-level changes
- ✅ Limited container impact if compromised

---

# ✍️ Task 5: Image Provenance (Signing)

## 🎯 Objective

Ensure image authenticity.

---

## 📌 Subtask 5.1: Sign Image

```bash
podman image sign --sign-by your@email.com nginx_nonroot
```

---

## 📌 Subtask 5.2: Verify Trust

```bash
podman image trust show
```

---

## 🔐 Key Concept

Image signing ensures:

- ✔ Image integrity
- ✔ Source verification
- ✔ Supply chain security

---

# ⚖️ Security Principles Summary

```text
Least Privilege Model
        │
        ▼
Minimal Base Image
        │
        ▼
No Root Access
        │
        ▼
Clean Packages
        │
        ▼
Signed Images
        │
        ▼
Secure Containers
```

---

# 🚨 Troubleshooting Tips

## ⚠️ Scan Fails

```bash
podman pull nginx:latest
```

Update Podman if needed.

---

## ⚠️ Permission Issues

```bash
sudo podman build .
```

---

## ⚠️ Alpine Package Issues

Search packages:

```bash
apk search nginx
```

---

## ⚠️ Signing Errors

Initialize GPG:

```bash
gpg --gen-key
```

---

# 📊 Best Practices

| Practice | Recommendation |
|----------|---------------|
| Use scan tools | ✅ Always |
| Use minimal images | ✅ Always |
| Run as root | ❌ Never |
| Remove cache | ✅ Always |
| Sign images | ✅ Recommended |
| Use latest blindly | ❌ Avoid |

---

# 🎉 Conclusion

In this lab, you successfully learned:

- 🔍 Image vulnerability scanning
- 🧹 Cache cleanup for smaller images
- 🪶 Using Alpine for minimal base images
- 👤 Running containers as non-root users
- ✍️ Image signing for trust verification

---

# 🔐 Final Takeaway

> Security in containers is not one step — it is a layered approach.

Each improvement reduces risk and strengthens your infrastructure.

---

# 🚀 Next Steps

- 🔹 Integrate scanning into CI/CD pipelines
- 🔹 Learn SELinux container policies
- 🔹 Explore Podman security flags
- 🔹 Use Kubernetes Pod Security Standards
- 🔹 Implement runtime security tools

---

# 🧹 Cleanup

```bash
podman stop secure_nginx

podman rm secure_nginx

podman rmi nginx nginx_clean nginx_alpine nginx_nonroot
```

---

# 📚 Additional Resources

- https://docs.podman.io/
- https://owasp.org/www-project-docker-top-10/
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/

---

<div align="center">
  
### 🔐 You Have Mastered Container Security Fundamentals

**Happy Secure Containerizing! 🚀**

</div>
