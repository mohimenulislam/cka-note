# kube-proxy

**kube-proxy** is a network proxy that runs on **every node** in a Kubernetes cluster.

It is responsible for implementing Kubernetes **Service networking** by forwarding traffic from a Service to its backend Pods.

> **Important:** A Kubernetes **Service** is a virtual object. It does **not** run as a Pod or container and has no network interface of its own. The kube-proxy makes Services accessible by configuring network rules on each node.

---

# Why Do We Need kube-proxy?

Pods are ephemeral.

Their IP addresses can change whenever they are recreated.

Example:

```
Database Pod

IP: 10.32.0.15
```

If the Pod is deleted and recreated:

```
Database Pod

IP: 10.32.0.20
```

Applications connecting directly to the Pod IP would fail.

Instead, Kubernetes provides a **Service** with a stable IP and DNS name.

Example:

```
Service Name : database-service

ClusterIP : 10.96.0.12
```

Applications communicate with the Service instead of the Pod.

---

# How Service Communication Works

```
Web Pod
     │
     ▼
database-service
(ClusterIP: 10.96.0.12)
     │
     ▼
Database Pod
(IP: 10.32.0.15)
```

The Web Pod does **not** know the actual Pod IP.

It only sends requests to the Service.

The Service forwards traffic to one of its backend Pods.

---

# What Does kube-proxy Do?

Whenever a new Service is created:

1. kube-proxy detects the new Service.
2. It creates the required networking rules on every node.
3. Those rules forward traffic from the Service IP to the backend Pods.

Example:

```
Client Request

10.96.0.12:80
      │
      ▼
kube-proxy Rule
      │
      ▼
10.32.0.15:80
```

---

# kube-proxy Workflow

```
Service Created
       │
       ▼
API Server
       │
       ▼
kube-proxy
(on every node)
       │
       ▼
Configure Network Rules
       │
       ▼
Forward Traffic
       │
       ▼
Backend Pods
```

---

# How kube-proxy Implements Services

One common implementation is using **iptables**.

Example:

```
Incoming Request

10.96.0.12
      │
      ▼
iptables Rule
      │
      ▼
10.32.0.15
```

The client never communicates directly with the Pod.

Instead, traffic is redirected by the rules created by kube-proxy.

> Modern Kubernetes clusters may also use **IPVS** mode for improved performance, but the purpose remains the same.

---

# kube-proxy Runs on Every Node

Unlike the API Server or Scheduler, kube-proxy runs on **every worker node**.

```
Cluster

Control Plane

Worker-1
 └── kube-proxy

Worker-2
 └── kube-proxy

Worker-3
 └── kube-proxy
```

This ensures that Services are reachable from any node in the cluster.

---

# kube-proxy Installation

If Kubernetes is installed manually:

1. Download the kube-proxy binary.
2. Configure it.
3. Run it as a system service.

---

# kubeadm Installation

When Kubernetes is installed using **kubeadm**, kube-proxy is deployed automatically as a **DaemonSet**.

A DaemonSet ensures that **one kube-proxy Pod runs on every node**.

View kube-proxy Pods:

```bash
kubectl get pods -n kube-system
```

Example:

```text
NAME                         READY   STATUS
kube-proxy-lzt6f             1/1     Running
kube-proxy-zm5qd             1/1     Running
```

---

# Verify kube-proxy DaemonSet

```bash
kubectl get daemonset -n kube-system
```

Example:

```text
NAME          DESIRED   CURRENT   READY

kube-proxy    3         3         3
```

---

# kube-proxy vs Service

| Component | Responsibility |
|-----------|----------------|
| Service | Provides a stable virtual IP and DNS name |
| kube-proxy | Configures networking rules to forward traffic |
| Backend Pod | Processes the application request |

---

# End-to-End Traffic Flow

```text
Client
   │
   ▼
Service
(ClusterIP)
   │
   ▼
kube-proxy
   │
   ▼
iptables / IPVS Rules
   │
   ▼
Backend Pod
```

---

# Overall Workflow

```text
Pod Created
      │
      ▼
Service Created
      │
      ▼
API Server
      │
      ▼
kube-proxy
(on every node)
      │
      ▼
Configure iptables/IPVS Rules
      │
      ▼
Forward Traffic
      │
      ▼
Backend Pod
```

---

# Key Points

- **kube-proxy** is a network proxy that runs on **every node**.
- It enables Kubernetes **Service networking**.
- It watches the API Server for Service and Endpoint changes.
- It configures **iptables** or **IPVS** rules to forward traffic.
- It does **not** process application traffic itself; it only sets up the forwarding rules.
- When installed with **kubeadm**, kube-proxy is deployed automatically as a **DaemonSet**, ensuring one Pod runs on each node.
