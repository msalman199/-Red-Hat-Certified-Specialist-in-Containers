# 🐳 Using RUN Instruction Efficiently 

<div align="center">

# 🚀 Using RUN Instruction Efficiently

### Learn How to Optimize Container Images Using the RUN Instruction

![Podman](https://img.shields.io/badge/Podman-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Containerfile](https://img.shields.io/badge/Containerfile-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![DevOps](https://img.shields.io/badge/DevOps-0A66C2?style=for-the-badge&logo=githubactions&logoColor=white)
![Containers](https://img.shields.io/badge/Containers-2496ED?style=for-the-badge&logo=docker&logoColor=white)

</div>

---

# 📖 Overview

The **RUN** instruction is one of the most important instructions in a Dockerfile or Containerfile.

Every `RUN` instruction creates a new image layer. Poorly designed RUN commands can lead to:

- ❌ Larger image sizes
- ❌ Slower builds
- ❌ More storage usage
- ❌ Longer deployment times

This lab demonstrates how to efficiently use the RUN instruction to create optimized and production-ready container images.

---

# 🎯 Objectives

By the end of this lab, you will be able to:

- ✅ Use RUN in Shell Form
- ✅ Use RUN in Exec Form
- ✅ Understand image layering
- ✅ Combine commands efficiently
- ✅ Reduce image size
- ✅ Analyze image layers using Podman
- ✅ Build optimized container images

---

# 📋 Prerequisites

Before starting this lab, ensure you have:

| Requirement | Description |
|------------|------------|
| 🐳 Podman or Docker | Installed and configured |
| 💻 Linux System | Ubuntu, CentOS, Fedora, RHEL |
| 🌐 Internet Access | Required for package downloads |
| 📘 Basic Container Knowledge | Dockerfiles & Containers |

---

# 🏗️ Understanding Image Layers

Every RUN instruction creates a separate image layer.

Example:

```dockerfile
RUN apt-get update
RUN apt-get install -y nginx
RUN rm -rf /var/lib/apt/lists/*
```

Produces:

```text
Layer 1 → apt-get update
Layer 2 → install nginx
Layer 3 → remove cache
```

More layers = Larger image size.

---

# 🚀 Task 1: Understanding RUN Instruction Forms

## 📌 Subtask 1.1: RUN in Shell Form

The shell form executes commands using:

```text
/bin/sh -c
```

Create a Dockerfile:

```dockerfile
FROM alpine:latest

RUN apk add --no-cache curl
```

### 🏗️ Build the Image

```bash
podman build -t run-lab-shell .
```

### Expected Outcome

✅ Alpine image downloaded

✅ curl installed

✅ New layer created

---

## 📚 Understanding Shell Form

Shell form syntax:

```dockerfile
RUN command
```

Example:

```dockerfile
RUN echo "Hello World"
```

### Advantages

- ✅ Simple syntax
- ✅ Easy to read
- ✅ Supports shell features
- ✅ Supports command chaining

Example:

```dockerfile
RUN echo "Hello" && echo "World"
```

---

## 📌 Subtask 1.2: RUN in Exec Form

The exec form avoids shell processing.

Modify the Dockerfile:

```dockerfile
FROM alpine:latest

RUN ["/bin/sh", "-c", "apk add --no-cache curl"]
```

### 🏗️ Build the Image

```bash
podman build -t run-lab-exec .
```

### Expected Outcome

✅ curl installed successfully

✅ Image builds normally

---

## 📚 Understanding Exec Form

Syntax:

```dockerfile
RUN ["executable", "parameter1", "parameter2"]
```

Example:

```dockerfile
RUN ["echo", "Hello World"]
```

### Advantages

- ✅ Avoids shell parsing issues
- ✅ Better argument handling
- ✅ More predictable behavior
- ✅ Preferred for complex commands

---

# ⚖️ Shell Form vs Exec Form

| Feature | Shell Form | Exec Form |
|----------|-----------|------------|
| Easy Syntax | ✅ | ⚠️ |
| Shell Expansion | ✅ | ❌ |
| Variable Expansion | ✅ | ❌ |
| Predictable Execution | ⚠️ | ✅ |
| JSON Format | ❌ | ✅ |

---

# 🚀 Task 2: Combining Commands to Minimize Layers

## 📌 Subtask 2.1: Inefficient Multi-Layer Approach

Example of an inefficient Dockerfile:

```dockerfile
FROM ubuntu:latest

RUN apt-get update

RUN apt-get install -y nginx

RUN rm -rf /var/lib/apt/lists/*
```

### ❌ Problem

Each RUN creates a layer:

```text
Layer 1 → apt-get update

Layer 2 → install nginx

Layer 3 → remove cache
```

Result:

- Larger image
- More layers
- Slower downloads
- Less efficient builds

---

## 📌 Subtask 2.2: Optimized Single-Layer Approach

Combine commands into one RUN statement:

```dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y nginx && \
    rm -rf /var/lib/apt/lists/*
```

### ✅ Why This Is Better

Only one layer is created:

```text
Layer 1
 ├── Update packages
 ├── Install nginx
 └── Remove cache
```

Benefits:

- ✅ Smaller image
- ✅ Faster build
- ✅ Cleaner image
- ✅ Better production practice

---

### 🏗️ Build Optimized Image

```bash
podman build -t optimized-nginx .
```

---

### 🔍 Compare Images

```bash
podman images | grep -E 'optimized-nginx|run-lab-shell'
```

Expected Result:

```text
optimized-nginx
run-lab-shell
```

The optimized image should contain fewer layers and less wasted space.

---

# 🚀 Task 3: Verifying Layer Reduction

## 📌 Subtask 3.1: Inspecting Image Layers

Analyze image history:

```bash
podman history optimized-nginx
```

### Expected Output

```text
IMAGE        CREATED      CREATED BY
abc123       2 mins ago   RUN apt-get update && ...
```

### Output Analysis

Notice:

- ✅ One combined RUN layer
- ✅ Fewer image layers
- ✅ Cleaner image structure

---

## 📌 Subtask 3.2: Comparing Image Sizes

View images:

```bash
podman images
```

### Example Output

```text
REPOSITORY         TAG       SIZE

optimized-nginx    latest    130MB

run-lab-shell      latest    150MB
```

---

# 🔥 Key Takeaway

```text
Fewer Layers
      ↓

Smaller Images
      ↓

Faster Downloads
      ↓

Faster Deployments
      ↓

Better Production Containers
```

---

# 🏗️ Optimization Workflow

```text
Write Dockerfile
        │
        ▼

Multiple RUN Commands
        │
        ▼

Analyze Layers
        │
        ▼

Combine Commands
        │
        ▼

Remove Cache Files
        │
        ▼

Rebuild Image
        │
        ▼

Verify Optimization
```

---

# 🚨 Troubleshooting Tips

## ⚠️ Permission Issues

If Podman encounters permission errors:

```bash
sudo podman build -t optimized-nginx .
```

Or configure rootless Podman correctly.

---

## ⚠️ Package Updates Not Reflected

Force a clean build:

```bash
podman build --no-cache -t optimized-nginx .
```

---

## ⚠️ Build Failures

Verify Dockerfile syntax:

```bash
cat Dockerfile
```

Check:

- Missing backslashes
- Invalid package names
- Incorrect RUN syntax

---

## ⚠️ Internet Connectivity Issues

Test connectivity:

```bash
ping google.com
```

Verify registry access:

```bash
podman pull alpine
```

---

# 📚 Best Practices for RUN

| Practice | Recommendation |
|-----------|---------------|
| Combine Commands | ✅ Always |
| Remove Package Cache | ✅ Always |
| Use Multiple RUNs | ❌ Avoid |
| Use && Between Commands | ✅ Recommended |
| Analyze Layers | ✅ Use podman history |
| Use Minimal Images | ✅ Recommended |

---

# 🎉 Conclusion

In this lab, you successfully:

✅ Used RUN in Shell Form

✅ Used RUN in Exec Form

✅ Learned how image layers work

✅ Combined commands efficiently

✅ Reduced image size

✅ Inspected image history

✅ Verified optimization using Podman

Efficient use of the RUN instruction is one of the most important container optimization techniques. Combining package installation and cleanup into a single layer results in smaller, faster, and more secure container images.

---

# 🚀 Next Steps

Continue your container optimization journey:

- 🔹 Learn COPY vs ADD
- 🔹 Explore Multi-Stage Builds
- 🔹 Use .dockerignore Files
- 🔹 Optimize Layer Caching
- 🔹 Analyze Images with podman history
- 🔹 Build Production-Ready Containers

---

# 🧹 Cleanup (Optional)

## Remove Created Images

```bash
podman rmi optimized-nginx

podman rmi run-lab-shell

podman rmi run-lab-exec
```

---

## Remove Unused Images

```bash
podman image prune -a
```

---

# 📚 Additional Resources

### Podman Documentation

https://docs.podman.io/

### Dockerfile Best Practices

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

### Containerfile Reference

https://docs.docker.com/engine/reference/builder/

---

<div align="center">
### 🐳 Mastering the RUN Instruction!

**Happy Learning & Happy Containerizing! 🚀**

</div>
