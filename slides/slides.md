---
marp: true
paginate: true
theme: my-theme
---

# Microservices & Containerisation met .NET Aspire
 
---

![bg left:40%](img/me.jpeg)

## Rick Neeft

### Developer @VECOZO

<i class="bi bi-linkedin"></i> LinkedIn: [rneeft](https://linkedin.com/in/rneeft)
<i class="bi bi-github"></i> Blog: [rickneeft.dev](https://www.rickneeft.dev)
<i class="bi bi-browser-chrome"></i> GitHub: [rneeft](https://github.com/rneeft)

---

# Agenda

- Introduction
- Case
- Give it a go
- .NET Aspire
- Identity Provider
- Messaging

---

# Course

```
https://www.rickneeft.dev/aspire-course-site/
```

```
https://github.com/rneeft/workshop-vecozo-26-mei
```

---


# Domein COV

---

# Introduction to Docker

- Docker is an open platform for developing, shipping, and running applications.
- It uses containerization to package applications and their dependencies.
- Containers are lightweight and portable.

---

# Why Docker?

- Consistency across environments
- Lightweight and fast startup
- Easier CI/CD pipelines
- Better resource utilization

---

# Docker vs Virtual Machines

## Virtual Machines
- Emulate entire hardware systems
- Include a full OS
- Heavy resource usage
- Slow to boot

## Docker Containers
- Share the host OS kernel
- Contain only application + dependencies
- Lightweight
- Fast to start

---

# Architecture Comparison

```
VM:
| Hardware |
| Host OS |
| Hypervisor |
| Guest OS |
| App |

Docker:
| Hardware |
| Host OS |
| Docker Engine |
| Container (App + Dependencies) |
```

---

# What is a Dockerfile?

- A text file with instructions to build a Docker image.
- Defines the base image, dependencies, commands, etc.

### Example Dockerfile
```Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "index.js"]
```

---

# What is Docker Compose?

- A tool for defining and running multi-container Docker applications.
- Uses a YAML file (`docker-compose.yml`).
- Manages services, networks, and volumes in one config.

### Example docker-compose.yml
```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "8080:8080"
  db:
    image: postgres
```

---

# Benefits of Docker Compose

- Simplifies configuration for multi-container setups
- Easy to scale services
- Useful for local development and testing

---

# Summary

- Docker simplifies app deployment with containers
- More efficient than virtual machines
- Dockerfiles define how to build images
- Docker Compose manages multi-container applications

---

# Thank You!

_Questions?_
