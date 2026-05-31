# 🌍 Using ENV and Environment Variables with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Linux-Alpine-orange?style=for-the-badge&logo=alpinelinux" />
  <img src="https://img.shields.io/badge/Containerfile-ENV-blue?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/DevOps-Configuration-green?style=for-the-badge&logo=redhat" />
  <img src="https://img.shields.io/badge/OpenShift-Ready-red?style=for-the-badge&logo=redhatopenshift" />
</p>

---

# 📖 Overview

Environment variables are one of the most common methods for configuring containerized applications. They allow developers and DevOps engineers to customize application behavior without modifying source code.

In this lab, you will learn how to:

✅ Define environment variables inside a Containerfile

✅ Override variables at runtime using Podman

✅ Create multi-line environment variables

✅ Document environment variables for users and administrators

✅ Apply container configuration best practices

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 Define environment variables in a Containerfile

🔹 Override environment variables at runtime using `podman run -e`

🔹 Implement multi-line environment variables

🔹 Document environment variables effectively

🔹 Understand container configuration best practices

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement        | Description                       |
| ------------------ | --------------------------------- |
| 🐳 Podman          | Version 3.0+ recommended          |
| 🐧 Linux Knowledge | Basic command-line experience     |
| 📝 Text Editor     | Nano, Vim, VS Code                |
| 🌐 Internet Access | Required to pull container images |

---

# 🏗️ Lab Setup

## 🔹 Verify Podman Installation

```bash
podman --version
```

### Expected Output

```bash
podman version 3.4.0
```

---

## 🔹 Create Working Directory

```bash
mkdir env-lab
cd env-lab
```

---

# 📌 Task 1: Setting ENV in Containerfile

---

## 🟢 Subtask 1.1: Create a Basic Containerfile

Create a new Containerfile:

```bash
nano Containerfile
```

Add the following content:

```dockerfile
FROM alpine:latest

ENV APP_NAME="MyApp" \
    APP_VERSION="1.0.0"

CMD echo "Running $APP_NAME version $APP_VERSION"
```

---

## 💡 Understanding ENV

The `ENV` instruction:

✔ Creates environment variables inside the image

✔ Makes variables available to applications

✔ Persists variables across container runs

✔ Helps separate configuration from code

Example:

```dockerfile
ENV APP_NAME="MyApp"
```

---

## 🟢 Subtask 1.2: Build and Run the Container

### Build the Image

```bash
podman build -t env-demo .
```

---

### Run the Container

```bash
podman run env-demo
```

### Expected Output

```text
Running MyApp version 1.0.0
```

---

# 📌 Task 2: Overriding ENV at Runtime

One of the biggest advantages of environment variables is that they can be changed without rebuilding the image.

---

## 🟢 Subtask 2.1: Override a Single Variable

Run the container:

```bash
podman run -e APP_NAME="NewApp" env-demo
```

### Expected Output

```text
Running NewApp version 1.0.0
```

---

## 🟢 Subtask 2.2: Override Multiple Variables

Run:

```bash
podman run \
-e APP_NAME="ProductionApp" \
-e APP_VERSION="2.0.0" \
env-demo
```

### Expected Output

```text
Running ProductionApp version 2.0.0
```

---

## 📊 Runtime Variable Priority

```text
Containerfile ENV
        │
        ▼
Runtime -e Variable
        │
        ▼
Runtime Value Wins
```

Example:

```dockerfile
ENV APP_NAME="MyApp"
```

Runtime:

```bash
podman run -e APP_NAME="ProductionApp"
```

Result:

```text
ProductionApp
```

---

# 📌 Task 3: Using Multi-line ENV

Multi-line environment variables are useful for:

✅ Descriptions

✅ Configuration templates

✅ Long messages

✅ Documentation

---

## 🟢 Subtask 3.1: Create Multi-line Variables

Edit your Containerfile:

```dockerfile
FROM alpine:latest

ENV APP_NAME="MyApp" \
    APP_VERSION="1.0.0" \
    APP_DESCRIPTION="This is a multi-line \
environment variable example"

CMD echo "$APP_DESCRIPTION"
```

---

### Build the Image

```bash
podman build -t multiline-env .
```

---

### Run the Container

```bash
podman run multiline-env
```

---

### Expected Output

```text
This is a multi-line environment variable example
```

---

## 💡 Multi-line ENV Best Practices

✔ Use backslashes (`\`) correctly

✔ Keep variables readable

✔ Avoid unnecessary line breaks

✔ Group related variables together

Example:

```dockerfile
ENV DATABASE_HOST="db" \
    DATABASE_PORT="5432" \
    DATABASE_NAME="production"
```

---

# 📌 Task 4: Documenting Environment Variables

Good documentation helps:

✅ Developers

✅ System Administrators

✅ DevOps Engineers

✅ OpenShift/Kubernetes Teams

---

## 🟢 Subtask 4.1: Create Documentation

Create README.md:

```bash
nano README.md
```

Add:

```markdown
# Environment Variables

| Variable | Description | Default Value |
|-----------|-------------|---------------|
| APP_NAME | Name of the application | MyApp |
| APP_VERSION | Application version | 1.0.0 |
| APP_DESCRIPTION | Multi-line description | See Containerfile |
```

---

## 📋 Example Documentation Table

| Variable        | Description             | Default Value     |
| --------------- | ----------------------- | ----------------- |
| APP_NAME        | Name of the application | MyApp             |
| APP_VERSION     | Application version     | 1.0.0             |
| APP_DESCRIPTION | Multi-line description  | See Containerfile |

---

## 🟢 Subtask 4.2: Verify Documentation

Confirm:

✔ Every ENV variable is documented

✔ Default values are accurate

✔ Descriptions are meaningful

✔ Documentation matches the Containerfile

---

# 🔍 Inspecting Environment Variables

View container configuration:

```bash
podman inspect <container>
```

Example:

```bash
podman inspect env-demo
```

---

### Filter Environment Variables

```bash
podman inspect env-demo | grep APP_
```

---

# ⚠️ Troubleshooting Guide

## Problem 1: Variables Not Updating

### Incorrect

```bash
podman run APP_NAME="Test"
```

### Correct

```bash
podman run -e APP_NAME="Test" env-demo
```

---

## Problem 2: Multi-line Variable Errors

### Incorrect

```dockerfile
ENV APP_DESCRIPTION="This is
a test"
```

### Correct

```dockerfile
ENV APP_DESCRIPTION="This is \
a test"
```

---

## Problem 3: Variable Not Visible

Inspect container:

```bash
podman inspect <container>
```

Check:

```bash
podman exec -it <container> env
```

---

# 🧪 Final Verification

---

## Build Image

```bash
podman build -t env-demo .
```

---

## Run Default Configuration

```bash
podman run env-demo
```

Expected:

```text
Running MyApp version 1.0.0
```

---

## Run Custom Configuration

```bash
podman run \
-e APP_NAME="ProductionApp" \
-e APP_VERSION="2.0.0" \
env-demo
```

Expected:

```text
Running ProductionApp version 2.0.0
```

---

## Verify Environment Variables

```bash
podman inspect env-demo
```

---

# 🧹 Cleanup

Remove containers:

```bash
podman rm -a
```

---

Remove images:

```bash
podman rmi env-demo multiline-env
```

---

Verify cleanup:

```bash
podman images
```

---

# 🚀 Real-World DevOps Use Cases

Environment variables are widely used for:

🔹 Database Connections

🔹 API Endpoints

🔹 Application Configuration

🔹 Feature Flags

🔹 Deployment Environments

🔹 OpenShift Configurations

🔹 Kubernetes Deployments

Example:

```bash
podman run \
-e DATABASE_HOST=db.example.com \
-e DATABASE_PORT=5432 \
-e ENVIRONMENT=production \
myapp
```

---

# 📚 Additional Learning

Explore:

### Container Configuration

```bash
man podman-run
```

### Environment Variables

```bash
man env
```

### Podman Documentation

https://docs.podman.io

### OpenShift Documentation

https://docs.openshift.com

---

# 🎓 Lab Summary

In this lab, you successfully learned how to:

✅ Define environment variables using `ENV`

✅ Override values at runtime using `-e`

✅ Create multi-line environment variables

✅ Document container configuration

✅ Inspect container environments

✅ Apply container configuration best practices

✅ Prepare applications for OpenShift and Kubernetes deployments

---



🎉 Congratulations!

You have completed the **Using ENV and Environment Variables with Podman** lab.

You now understand one of the most important configuration techniques used in modern Cloud, DevOps, Kubernetes, OpenShift, and Containerized Application deployments.

🚀 Happy Learning and Happy Containerizing!
