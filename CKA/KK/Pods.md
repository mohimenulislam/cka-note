# Kubernetes Pods

A **Pod** is the **smallest deployable unit** in Kubernetes.

A Pod is a wrapper around one or more containers that share the same networking and storage resources.

> **Important:** Kubernetes **does not deploy containers directly**. It always deploys containers inside a **Pod**.

---

# Prerequisites

Before deploying Pods, ensure:

- ✅ Your application is already packaged as a Docker image.
- ✅ The image is available in a container registry (Docker Hub or a private registry).
- ✅ Your Kubernetes cluster is up and running.
- ✅ All Kubernetes components are healthy.

---

# Deploying a Pod

The easiest way to create a Pod is:

```bash
kubectl run nginx --image=nginx
```

This command:

- Creates a Pod named **nginx**
- Pulls the **nginx** image from Docker Hub
- Starts one container inside the Pod

Verify the Pod:

```bash
kubectl get pods
```

Example:

```text
NAME    READY   STATUS              AGE
nginx   0/1     ContainerCreating   5s
```

After a few seconds:

```text
NAME    READY   STATUS    AGE
nginx   1/1     Running   15s
```

---

# Where Does the Image Come From?

The `--image` parameter specifies the container image.

Example:

```bash
kubectl run nginx --image=nginx
```

Here:

- Image Name: `nginx`
- Registry: Docker Hub (default)

Kubernetes downloads the image if it is not already present on the node.

Images can also be pulled from:

- Docker Hub
- Google Container Registry (GCR)
- Amazon ECR
- Azure Container Registry (ACR)
- GitHub Container Registry (GHCR)
- Private registries

---

# Pod Lifecycle

```
kubectl run
      │
      ▼
Create Pod
      │
      ▼
Scheduler selects a node
      │
      ▼
kubelet receives Pod
      │
      ▼
Pull Container Image
      │
      ▼
Create Container
      │
      ▼
Running
```

---

# Scaling an Application

Suppose one Pod is running:

```
nginx Pod
```

If more users access the application, **do not add more containers to the same Pod**.

Instead, create **additional Pods**.

Example:

```
Before

Pod-1
 └── nginx
```

Scale out:

```
Pod-1
 └── nginx

Pod-2
 └── nginx

Pod-3
 └── nginx
```

> **Important:** Kubernetes scales applications by **adding Pods**, **not by adding containers to an existing Pod**.

---

# One Pod vs Multiple Pods

### ❌ Incorrect

```
Pod

├── nginx
├── nginx
├── nginx
```

### ✅ Correct

```
Pod-1
 └── nginx

Pod-2
 └── nginx

Pod-3
 └── nginx
```

Each Pod is an independent instance of the application.

---

# Multi-Container Pods

Although a Pod usually contains **one container**, it can contain **multiple containers**.

Example:

```
Pod

├── Web Application
└── Log Collector
```

or

```
Pod

├── Main Application
└── Helper Container
```

The helper container performs supporting tasks such as:

- Log collection
- Monitoring
- Data processing
- Proxying
- File synchronization

> **Important:** Multi-container Pods are **less common** than single-container Pods and are typically used when the containers must work closely together.

---

# Why Use Multiple Containers in One Pod?

Containers inside the same Pod:

- Share the same network namespace.
- Share the same IP address.
- Communicate using `localhost`.
- Can share storage volumes.
- Are created together.
- Are stopped together.

Example:

```
Pod

+----------------------------+

Main App
127.0.0.1

Helper Container
127.0.0.1

Shared Volume

+----------------------------+
```

> **Important:** Containers within the same Pod communicate over **localhost**, not through a Service.

---

# Why Not Use Plain Docker?

Imagine deploying an application directly using Docker.

```
docker run app
```

Later, the application becomes more complex:

```
Application

+
Helper Container

+
Monitoring Container

+
Log Collector
```

You would need to manually manage:

- Networking
- Shared volumes
- Startup order
- Container communication
- Container lifecycle
- Restarts
- Cleanup

Kubernetes Pods automatically handle these relationships by grouping related containers together.

---

# Pod Networking

Every Pod receives its own IP address.

Example:

```
Pod-1

IP: 10.244.1.5
```

Another Pod:

```
Pod-2

IP: 10.244.2.8
```

Pods communicate using their Pod IPs or, more commonly, through Kubernetes **Services**.

---

# Pod Status

Common Pod states:

| Status | Description |
|---------|-------------|
| `Pending` | Pod accepted but not yet running |
| `ContainerCreating` | Container image is being prepared |
| `Running` | Pod is running successfully |
| `Succeeded` | Pod completed successfully |
| `Failed` | Pod terminated with an error |
| `CrashLoopBackOff` | Container repeatedly crashes and restarts |

---

# Useful Commands

Create a Pod:

```bash
kubectl run nginx --image=nginx
```

List Pods:

```bash
kubectl get pods
```

Get detailed information:

```bash
kubectl describe pod nginx
```

Delete a Pod:

```bash
kubectl delete pod nginx
```

---

# Pod Architecture

```text
                Kubernetes Cluster
                        │
                        ▼
                 Worker Node
                        │
        ┌──────────────────────────┐
        │           Pod            │
        │                          │
        │  nginx Container         │
        └──────────────────────────┘
```

---

# Single vs Multi-Container Pod

### Single-Container Pod (Most Common)

```text
Pod
└── Application
```

### Multi-Container Pod

```text
Pod
├── Application
├── Logging Sidecar
└── Monitoring Agent
```

---

# Key Points

> **⭐ Kubernetes never deploys a container directly. Every container runs inside a Pod.**

> **⭐ A Pod is the smallest deployable unit in Kubernetes.**

> **⭐ A Pod usually contains one container, but it can contain multiple tightly coupled containers.**

> **⭐ Containers within the same Pod share the same IP address, network namespace, and storage volumes.**

> **⭐ To scale an application, create additional Pods—not additional containers in the same Pod.**

> **⭐ `kubectl run --image=<image>` creates a Pod and starts a container using the specified image.**

> **⭐ A newly created Pod typically transitions through `Pending` → `ContainerCreating` → `Running`.**
