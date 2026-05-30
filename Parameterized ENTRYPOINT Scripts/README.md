# ⚙️ Parameterized ENTRYPOINT Scripts in Containers

<div align="center">

# 🚀 Dynamic Container Initialization 

### Learn Argument Handling, Environment Modes & ENTRYPOINT Scripting

![Podman](https://img.shields.io/badge/Podman-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Shell](https://img.shields.io/badge/Shell_Scripting-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white)
![DevOps](https://img.shields.io/badge/DevOps-0A66C2?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

# 📖 Overview

Modern containers must adapt to different environments and runtime inputs.

This is achieved using:

- 🚀 ENTRYPOINT scripts
- ⚙️ Command-line arguments
- 🌍 Environment variables
- 🔄 Runtime execution logic

This lab teaches how to build flexible, production-ready container entrypoints.

---

# 🎯 Objectives

By the end of this lab, you will be able to:

- ✅ Create ENTRYPOINT scripts
- ✅ Pass and process runtime arguments
- ✅ Use CMD as default fallback commands
- ✅ Implement environment-based logic (dev/prod)
- ✅ Build command-driven containers
- ✅ Handle production-ready initialization flows

---

# 📋 Prerequisites

| Requirement | Description |
|------------|------------|
| 🐳 Podman/Docker | Installed |
| 💻 Linux system | Ubuntu / Fedora / RHEL |
| 📝 Editor | VS Code / Vim / Nano |
| 🧠 Shell knowledge | Basic Bash scripting |

---

# ⚙️ Setup

## 🔍 Step 1: Verify Podman

```bash
podman --version
```

---

## 📁 Step 2: Create Working Directory

```bash
mkdir entrypoint-lab

cd entrypoint-lab
```

---

# 🚀 Task 1: Basic ENTRYPOINT Script

## 🎯 Objective

Understand container initialization using scripts.

---

## 📌 Subtask 1.1: Create Script

```bash
nano entrypoint.sh
```

```bash
#!/bin/bash

echo "Container starting..."
echo "Executing as user: $(whoami)"
echo "Current directory: $(pwd)"
```

---

## 📌 Subtask 1.2: Make Executable

```bash
chmod +x entrypoint.sh
```

---

## 📌 Subtask 1.3: Dockerfile

```dockerfile
FROM alpine:latest

COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

---

## 🏗️ Build Image

```bash
podman build -t entrypoint-demo .
```

---

## ▶️ Run Container

```bash
podman run entrypoint-demo
```

---

### ✅ Expected Output

```text
Container starting...
Executing as user: root
Current directory: /
```

---

# ⚙️ Task 2: Parameter Handling

## 🎯 Objective

Pass runtime arguments into ENTRYPOINT.

---

## 📌 Subtask 2.1: Update Script

```bash
#!/bin/bash

echo "Container starting with arguments: $@"
echo "First argument: ${1:-none}"
echo "Second argument: ${2:-none}"

exec "$@"
```

---

## 📌 Subtask 2.2: Dockerfile with CMD

```dockerfile
FROM alpine:latest

COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

CMD ["echo", "Default command executed"]
```

---

## ▶️ Default Run

```bash
podman run entrypoint-demo
```

---

### Output

```text
Container starting with arguments: echo Default command executed
First argument: echo
Second argument: Default command executed
Default command executed
```

---

## 🔄 Override CMD

```bash
podman run entrypoint-demo ls -l /
```

---

# 🌍 Task 3: Environment-Specific Logic

## 🎯 Objective

Enable dev/prod behavior switching.

---

## 📌 Subtask 3.1: Update Script

```bash
#!/bin/bash

if [ "$ENV_MODE" = "production" ]; then
    echo "PRODUCTION MODE: Strict settings applied"

elif [ "$ENV_MODE" = "development" ]; then
    echo "DEVELOPMENT MODE: Debug features enabled"

else
    echo "No ENV_MODE specified, running default mode"
fi

exec "$@"
```

---

## 📌 Subtask 3.2: Dockerfile

```dockerfile
FROM alpine:latest

COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

CMD ["echo", "Running container"]
```

---

## ▶️ Test Development Mode

```bash
podman run -e ENV_MODE=development entrypoint-demo
```

---

## ▶️ Test Production Mode

```bash
podman run -e ENV_MODE=production entrypoint-demo
```

---

## ▶️ Default Mode

```bash
podman run entrypoint-demo
```

---

# ⚡ Task 4: Command Pattern Handling

## 🎯 Objective

Build command-driven container behavior.

---

## 📌 Subtask 4.1: Advanced Script

```bash
#!/bin/bash

case "$1" in
    start)
        echo "Starting application with config: ${2:-default}"
        ;;

    stop)
        echo "Stopping application gracefully"
        ;;

    *)
        echo "Usage: $0 {start|stop} [config]"
        exit 1
esac
```

---

## ▶️ Subtask 4.2: Test Commands

```bash
podman run entrypoint-demo start production
```

```bash
podman run entrypoint-demo stop
```

```bash
podman run entrypoint-demo invalid
```

---

# ⚠️ Troubleshooting

## ❌ Permission Denied

```bash
chmod +x entrypoint.sh
```

---

## ❌ ENV not working

```bash
podman run -e ENV_MODE=production image
```

---

## ❌ Exec Format Error

- Ensure:

```bash
#!/bin/bash
```

- Fix line endings:

```bash
dos2unix entrypoint.sh
```

---

# 📊 Key Concepts Summary

| Feature | Purpose |
|--------|--------|
| ENTRYPOINT | Main execution logic |
| CMD | Default arguments |
| $@ | All arguments |
| ENV | Environment switching |
| exec | Replace shell process |

---

# 🎉 Conclusion

In this lab, you learned:

- 🚀 How ENTRYPOINT scripts work
- ⚙️ How to pass arguments dynamically
- 🌍 How to use environment-based logic
- 🔄 How CMD and ENTRYPOINT interact
- 🧠 How to build production-ready containers

---

# 🔐 Final Takeaway

```text
ENTRYPOINT = container logic engine
CMD = default configuration
ENV = runtime behavior switch
```

---

# 🚀 Next Steps

- 🔹 Build CI/CD-ready entrypoint scripts
- 🔹 Integrate config files (JSON/YAML)
- 🔹 Use in Kubernetes deployments
- 🔹 Combine with health checks
- 🔹 Explore multi-stage builds

---

# 🧹 Cleanup

```bash
podman rmi entrypoint-demo
rm -rf entrypoint-lab
```

---

<div align="center">
  
### ⚙️ You Can Now Build Dynamic Containers

**Happy Container Scripting! 🚀**

</div>
