# 🔐 Managing Environment Files and Secrets with Podman

<p align="center">
  <img src="https://img.shields.io/badge/Container-Podman-purple?style=for-the-badge&logo=podman" />
  <img src="https://img.shields.io/badge/Security-Secrets-red?style=for-the-badge&logo=securityscorecard" />
  <img src="https://img.shields.io/badge/Configuration-.env_Files-blue?style=for-the-badge&logo=dotenv" />
  <img src="https://img.shields.io/badge/Compose-Podman_Compose-green?style=for-the-badge&logo=docker" />
  <img src="https://img.shields.io/badge/DevOps-Best_Practices-orange?style=for-the-badge&logo=redhat" />
</p>

---

# 📖 Overview

Managing configuration and sensitive information securely is a critical aspect of modern containerized applications. Podman provides support for both **environment files (.env)** and **secrets management**, allowing applications to remain portable, secure, and production-ready.

In this lab, you will learn how to:

✅ Create and manage `.env` files

✅ Pass environment variables from files

✅ Create and use Podman secrets

✅ Securely inject secrets into containers

✅ Integrate secrets with Podman Compose

✅ Apply container security best practices

---

# 🎯 Objectives

By the end of this lab, you will be able to:

🔹 Create and use `.env` files

🔹 Pass environment variables using `--env-file`

🔹 Create and manage Podman secrets

🔹 Access secrets inside containers

🔹 Use secrets within Podman Compose

🔹 Separate configuration from application code

---

# 🛠️ Prerequisites

Before starting this lab, ensure you have:

| Requirement            | Description                            |
| ---------------------- | -------------------------------------- |
| 🐳 Podman              | Version 3.0+ Recommended               |
| 🐧 Linux System        | RHEL, Fedora, Rocky, AlmaLinux, Ubuntu |
| 💻 Text Editor         | Vim, Nano, VS Code                     |
| 📚 Linux CLI Knowledge | Basic command-line skills              |
| 🌐 Internet Access     | Optional for pulling images            |
| 🔧 Podman Compose      | Installed and configured               |

---

# 🏗️ Lab Setup

## 🔹 Create Working Directory

```bash
mkdir podman-secrets-lab
cd podman-secrets-lab
```

---

## 🔹 Verify Podman Installation

```bash
podman --version
```

Expected Output:

```bash
podman version 3.x.x
```

---

## 🔹 Verify Podman Compose

```bash
podman-compose version
```

If not installed:

```bash
pip install podman-compose
```

---

# 📌 Task 1: Working with Environment Files

---

# 🟢 Subtask 1.1: Create a .env File

Create a new `.env` file:

```bash
cat <<EOF > .env
APP_NAME=MySecureApp
DB_USER=admin
DB_PASS=SuperSecret123!
EOF
```

---

## Verify File Contents

```bash
cat .env
```

Expected Output:

```text
APP_NAME=MySecureApp
DB_USER=admin
DB_PASS=SuperSecret123!
```

---

## 💡 What is a .env File?

A `.env` file stores configuration values separately from application code.

Benefits:

✅ Easier configuration management

✅ Environment-specific settings

✅ Better portability

✅ Follows DevOps best practices

---

## Environment Configuration Flow

```text
.env File
    │
    ▼
Environment Variables
    │
    ▼
Container Runtime
    │
    ▼
Application
```

---

# 🟢 Subtask 1.2: Use Environment Variables in a Container

Run a container using the environment file:

```bash
podman run --rm \
--env-file=.env \
alpine env | grep -E 'APP_NAME|DB_'
```

---

## Expected Output

```text
APP_NAME=MySecureApp
DB_USER=admin
DB_PASS=SuperSecret123!
```

---

## Key Concept

The `--env-file` flag allows bulk loading of environment variables from a file.

Instead of:

```bash
-e APP_NAME=MySecureApp \
-e DB_USER=admin \
-e DB_PASS=SuperSecret123!
```

You can manage everything centrally in `.env`.

---

# 📌 Task 2: Working with Podman Secrets

---

# 🟢 Subtask 2.1: Create a Secret

Create a file containing sensitive data:

```bash
echo "TopSecretPassword" > db_password.txt
```

Create the Podman secret:

```bash
podman secret create \
db_password_secret \
db_password.txt
```

---

## List Available Secrets

```bash
podman secret ls
```

Expected Output:

```text
ID             NAME
xxxxxxxxxxxx   db_password_secret
```

---

## Why Use Secrets?

Avoid storing:

❌ Passwords in source code

❌ Credentials in images

❌ Sensitive values in Git repositories

Instead use:

✅ Podman Secrets

✅ Secret Management Platforms

✅ Vault Solutions

---

## Secret Storage Architecture

```text
Secret File
     │
     ▼
Podman Secret Store
     │
     ▼
Container Runtime
     │
     ▼
/run/secrets/
```

---

# 🟢 Subtask 2.2: Use Secret in a Container

Mount the secret inside a container:

```bash
podman run --rm \
--secret=db_password_secret \
alpine \
cat /run/secrets/db_password_secret
```

---

## Expected Output

```text
TopSecretPassword
```

---

## Secret Location

Inside the container:

```text
/run/secrets/db_password_secret
```

---

## Verify Secret Exists

```bash
podman run --rm \
--secret=db_password_secret \
alpine ls /run/secrets
```

Expected:

```text
db_password_secret
```

---

## Troubleshooting Tip

If permission issues occur:

```bash
sudo podman secret ls
```

or test using:

```bash
--privileged
```

for troubleshooting purposes only.

---

# 📌 Task 3: Podman Compose Integration

---

# 🟢 Subtask 3.1: Create docker-compose.yml

Create the compose file:

```bash
cat <<EOF > docker-compose.yml
version: '3.8'

services:
  webapp:
    image: alpine

    env_file:
      - .env

    secrets:
      - db_password_secret

    command: sh -c "echo \$APP_NAME && cat /run/secrets/db_password_secret"

secrets:
  db_password_secret:
    external: true
EOF
```

---

## Understanding the Compose File

### Environment Variables

```yaml
env_file:
  - .env
```

Loads variables automatically.

---

### Secrets

```yaml
secrets:
  - db_password_secret
```

Makes secrets available inside containers.

---

### External Secret

```yaml
external: true
```

Indicates the secret already exists outside the compose file.

---

# 🟢 Subtask 3.2: Deploy with Podman Compose

Launch the application:

```bash
podman-compose up
```

---

## Expected Output

```text
webapp_1 | MySecureApp
webapp_1 | TopSecretPassword
```

---

## Deployment Architecture

```text
            .env File
                 │
                 ▼

           APP_NAME

                 │
                 ▼

┌─────────────────────────┐
│     Podman Compose      │
└─────────────┬───────────┘
              │
      ┌───────┴────────┐
      ▼                ▼

 Environment      Secret Store
 Variables             │
                        ▼

                db_password_secret

                        │
                        ▼

                 Container App
```

---

# 📊 Environment Variables vs Secrets

| Feature                | Environment Variables | Podman Secrets |
| ---------------------- | --------------------- | -------------- |
| Easy Setup             | ✅                     | ⚠️             |
| Secure Storage         | ❌                     | ✅              |
| Visible in Inspect     | ✅                     | ❌              |
| Suitable for Passwords | ❌                     | ✅              |
| Configuration Data     | ✅                     | ⚠️             |
| Production Ready       | ⚠️                    | ✅              |

---

# 🔐 Security Best Practices

## Never Store

❌ Passwords in Containerfiles

❌ API Keys in Git

❌ Database Credentials in Images

❌ Secrets in Source Code

---

## Always Use

✅ Podman Secrets

✅ Secret Rotation

✅ Least Privilege Access

✅ External Secret Management

✅ Environment Separation

---

# 🧪 Final Verification

Verify environment file:

```bash
cat .env
```

---

Verify secret exists:

```bash
podman secret ls
```

---

Verify secret access:

```bash
podman run --rm \
--secret=db_password_secret \
alpine cat /run/secrets/db_password_secret
```

---

Verify compose deployment:

```bash
podman-compose up
```

---

# 🔍 Troubleshooting Guide

## Problem 1: Environment Variables Not Loading

Verify:

```bash
cat .env
```

Ensure format:

```text
KEY=value
```

No spaces around `=`.

---

## Problem 2: Secret Not Found

Check:

```bash
podman secret ls
```

Recreate if necessary:

```bash
podman secret create \
db_password_secret \
db_password.txt
```

---

## Problem 3: Compose Cannot Access Secret

Verify:

```yaml
external: true
```

matches the secret name exactly.

---

## Problem 4: Secret File Missing

Check:

```bash
ls /run/secrets
```

inside the container.

---

# 🚀 Real-World DevOps Use Cases

These techniques are widely used for:

🔹 Database Credentials

🔹 API Keys

🔹 TLS Certificates

🔹 Cloud Authentication Tokens

🔹 OpenShift Deployments

🔹 Kubernetes Secrets

🔹 CI/CD Pipelines

🔹 Production Container Security

---

# 📚 Additional Resources

## Podman Secret Documentation

```bash
man podman-secret
```

---

## Podman Compose Documentation

```bash
podman-compose --help
```

---

## Twelve-Factor Configuration Principles

https://12factor.net/config

---

## Podman Official Documentation

https://docs.podman.io

---

# 🧹 Cleanup

## Remove Secret

```bash
podman secret rm db_password_secret
```

---

## Remove Lab Directory

```bash
cd ..
rm -rf podman-secrets-lab
```

---

## Verify Cleanup

```bash
podman secret ls
```

Expected:

```text
No secrets found
```

---

# 🎓 Lab Summary

In this lab, you successfully learned how to:

✅ Create and manage `.env` files

✅ Load environment variables using `--env-file`

✅ Create Podman secrets

✅ Access secrets securely inside containers

✅ Integrate secrets with Podman Compose

✅ Apply secure container configuration practices

---

# 🏆 Completion Achievement

🎉 Congratulations!

You have completed the **Managing Environment Files and Secrets with Podman** lab.

You now possess essential configuration management and secret handling skills used by:

🚀 DevOps Engineers

🚀 Platform Engineers

🚀 OpenShift Administrators

🚀 Kubernetes Operators

🚀 Cloud Engineers

🚀 Site Reliability Engineers (SREs)

These skills are critical for building secure, production-ready containerized applications and cloud-native platforms.

Happy Learning and Happy Secure Containerizing! 🔐🚀
