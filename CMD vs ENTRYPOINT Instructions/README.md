# ⚙️ CMD vs ENTRYPOINT Instructions in Containers

<div align="center">

# 🚀 Container Command Control 

### Learn CMD, ENTRYPOINT, and Runtime Behavior in Docker/Podman

![Podman](https://img.shields.io/badge/Podman-892CA0?style=for-the-badge&logo=podman&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![DevOps](https://img.shields.io/badge/DevOps-0A66C2?style=for-the-badge&logo=githubactions&logoColor=white)
![Containers](https://img.shields.io/badge/Containers-FF6B6B?style=for-the-badge&logo=kubernetes&logoColor=white)

</div>

---

# 📖 Overview

Container behavior at runtime is controlled by two key instructions:

- ⚡ `CMD` → Default command (can be overridden)
- 🚀 `ENTRYPOINT` → Fixed executable behavior

Understanding these is essential for building flexible and production-ready containers.

---

# 🎯 Objectives

By the end of this lab, you will be able to:

- ✅ Understand CMD instruction behavior
- ✅ Understand ENTRYPOINT behavior
- ✅ Combine ENTRYPOINT + CMD correctly
- ✅ Override commands at runtime
- ✅ Build script-based entrypoints
- ✅ Debug container execution issues

---

# 📋 Prerequisites

| Requirement | Description |
|------------|------------|
| 🐳 Podman/Docker | Installed |
| 💻 Linux system | Ubuntu / Fedora / RHEL |
| 📝 Editor | Vim / Nano / VS Code |
| 🌐 Internet | Required for image pulls |

---

# ⚙️ Lab Setup

## 🔍 Step 1: Verify Podman

```bash
podman --version
```

---

## 📁 Step 2: Create Lab Directory

```bash
mkdir cmd-entrypoint-lab

cd cmd-entrypoint-lab
```

---

# ⚡ Task 1: Understanding CMD

## 🎯 Objective

Learn how CMD provides default execution behavior.

---

## 📌 Subtask 1.1: Create Containerfile

```dockerfile
FROM alpine:latest

CMD ["echo", "Hello from CMD"]
```

---

## 🏗️ Build Image

```bash
podman build -t cmd-demo -f Containerfile.cmd
```

---

## ▶️ Run Container

```bash
podman run cmd-demo
```

---

### ✅ Expected Output

```text
Hello from CMD
```

---

## 🔄 Subtask 1.2: Override CMD

```bash
podman run cmd-demo echo "Overridden command"
```

---

### Output

```text
Overridden command
```

---

## 📚 Key Concept

```text
CMD = default behavior (easily overridden)
```

---

# 🚀 Task 2: Understanding ENTRYPOINT

## 🎯 Objective

Make containers behave like executables.

---

## 📌 Subtask 2.1: ENTRYPOINT Setup

```dockerfile
FROM alpine:latest

ENTRYPOINT ["echo", "Hello from ENTRYPOINT"]
```

---

## 🏗️ Build Image

```bash
podman build -t entrypoint-demo -f Containerfile.entrypoint
```

---

## ▶️ Run Container

```bash
podman run entrypoint-demo
```

---

### Output

```text
Hello from ENTRYPOINT
```

---

## ➕ Subtask 2.2: Append Arguments

```bash
podman run entrypoint-demo "with appended text"
```

---

### Output

```text
Hello from ENTRYPOINT with appended text
```

---

## 📚 Key Concept

```text
ENTRYPOINT = fixed executable + appended arguments
```

---

# 🔗 Task 3: Combining ENTRYPOINT + CMD

## 🎯 Objective

Understand default arguments with flexibility.

---

## 📌 Subtask 3.1: Combined Setup

```dockerfile
FROM alpine:latest

ENTRYPOINT ["echo"]

CMD ["Default message"]
```

---

## 🏗️ Build Image

```bash
podman build -t combined-demo -f Containerfile.combined
```

---

## ▶️ Run Container

```bash
podman run combined-demo
```

---

### Output

```text
Default message
```

---

## 🔄 Subtask 3.2: Override CMD

```bash
podman run combined-demo "Custom message"
```

---

### Output

```text
Custom message
```

---

## 📚 Key Concept

```text
ENTRYPOINT = executable
CMD = default arguments
```

---

# 🧠 Task 4: Advanced Patterns

## 🎯 Objective

Use scripts and runtime flexibility.

---

## 📌 Subtask 4.1: Script EntryPoint

### Create script

```bash
nano entrypoint.sh
```

```sh
#!/bin/sh

echo "Starting container with arguments: $@"
exec "$@"
```

---

## 📌 Containerfile.script

```dockerfile
FROM alpine:latest

COPY entrypoint.sh /entrypoint.sh

RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

CMD ["echo", "Default execution"]
```

---

## 🏗️ Build Image

```bash
podman build -t script-demo -f Containerfile.script
```

---

## ▶️ Run Container

```bash
podman run script-demo
```

---

## 🔄 Subtask 4.2: Full Override

```bash
podman run --entrypoint /bin/ls script-demo -l /
```

---

### Output

Directory listing of `/`

---

## 📚 Key Concept

```text
ENTRYPOINT script = flexible runtime control layer
```

---

# ⚠️ Troubleshooting Tips

## ❌ Command Not Executing

- Check JSON format:
```json
["echo", "hello"]
```

---

## ❌ Script Errors

- Ensure:
```bash
chmod +x entrypoint.sh
```

---

## ❌ Exec Format Error

- Add shebang:
```bash
#!/bin/sh
```

---

## ❌ Line Ending Issues

- Use LF, not CRLF

---

# 📊 CMD vs ENTRYPOINT Comparison

| Feature | CMD | ENTRYPOINT |
|--------|-----|------------|
| Default command | ✅ | ❌ |
| Override easily | ✅ | ❌ (hard) |
| Fixed execution | ❌ | ✅ |
| Best use case | Defaults | Executables |

---

# 🎉 Conclusion

In this lab, you learned:

- ⚡ CMD provides default commands
- 🚀 ENTRYPOINT defines container execution
- 🔗 Combining both gives flexibility
- 🧠 Scripts enhance container behavior
- 🔄 Runtime overrides behave differently

---

# 🔐 Final Verification

## List Images

```bash
podman images
```

---

## Test Final Run

```bash
podman run --rm combined-demo "Lab completed successfully!"
```

---

# 🧹 Cleanup

```bash
podman rmi cmd-demo entrypoint-demo combined-demo script-demo
```

---

# 🚀 Next Steps

- 🔹 Learn Docker multi-stage builds
- 🔹 Use ENTRYPOINT in Kubernetes pods
- 🔹 Explore Helm charts behavior
- 🔹 Combine with health checks

---

<div align="center">

### ⚙️ You Now Understand Container Execution Control

**Happy Container Engineering! 🚀**

</div>
