# Kube Scheduler

The **Kube Scheduler** is responsible for **scheduling Pods onto worker nodes**.

When a Pod is created without a node assignment, the scheduler determines the most suitable node based on the Pod's resource requirements and scheduling constraints.

> **Important:** The scheduler **does not create or run Pods**. It only decides **which node** a Pod should run on. The **kubelet** on the selected node is responsible for creating and running the Pod.

---

# Why Do We Need a Scheduler?

Imagine a cluster with multiple worker nodes.

Each node has different available resources.

Example:

| Node | Available CPU |
|------|---------------|
| Node-1 | 4 CPU |
| Node-2 | 4 CPU |
| Node-3 | 12 CPU |
| Node-4 | 16 CPU |

Suppose a Pod requires:

```
CPU: 10
```

The scheduler must decide which node can run this Pod.

---

# Scheduler Workflow

When a new Pod is created:

```
User
   │
   ▼
API Server
   │
   ▼
Pod created (Pending)
   │
   ▼
Kube Scheduler
   │
   ▼
Select Best Node
   │
   ▼
API Server updates Pod
   │
   ▼
Kubelet creates the Pod
```

---

# Scheduling Process

The scheduler schedules Pods in **two phases**:

1. **Filtering**
2. **Scoring (Ranking)**

---

# Phase 1: Filter Nodes

The scheduler first filters out nodes that **cannot** run the Pod.

Example:

Pod requires:

```
CPU = 10
```

Cluster:

```
Node-1 → 4 CPU
Node-2 → 4 CPU
Node-3 → 12 CPU
Node-4 → 16 CPU
```

Filtering result:

```
❌ Node-1 (Not enough CPU)
❌ Node-2 (Not enough CPU)

✅ Node-3
✅ Node-4
```

Only nodes that satisfy the Pod requirements move to the next phase.

---

# Phase 2: Score (Rank) Nodes

After filtering, the scheduler ranks the remaining nodes.

It assigns each node a **score (0–10)** based on various priority functions.

Example:

After placing the Pod:

```
Node-3

12 CPU
-10 CPU
---------
2 CPU free
```

```
Node-4

16 CPU
-10 CPU
---------
6 CPU free
```

Since **Node-4** will have more free CPU remaining, it receives a higher score.

```
Node-3 → Score: 5

Node-4 → Score: 9
```

Result:

```
Pod scheduled to Node-4
```

---

# Scheduling Decision

```
            Pod (CPU:10)

                 │
                 ▼

     Filter Nodes
──────────────────────────

Node-1 (4 CPU)  ❌

Node-2 (4 CPU)  ❌

Node-3 (12 CPU) ✅

Node-4 (16 CPU) ✅

                 │
                 ▼

       Score Remaining Nodes

Node-3 → Score 5

Node-4 → Score 9

                 │
                 ▼

       Select Node-4
```

---

# Scheduler Decision Factors

The scheduler considers many factors before selecting a node.

Examples include:

- Resource requests (CPU & Memory)
- Resource limits
- Node Selector
- Node Affinity
- Pod Affinity
- Pod Anti-Affinity
- Taints & Tolerations
- Topology Spread Constraints
- Available resources
- Custom scheduler plugins

---

# kube-scheduler Installation

If Kubernetes is installed manually, download the scheduler binary from the Kubernetes release page and run it as a service.

---

# Viewing kube-scheduler (kubeadm)

When Kubernetes is installed using **kubeadm**, the scheduler runs as a **Static Pod**.

Configuration file:

```bash
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

---

# Viewing kube-scheduler Process

To verify that the scheduler is running:

```bash
ps -aux | grep kube-scheduler
```

Example output:

```text
root   1435   ... kube-scheduler
```

---

# Summary

| Component | Responsibility |
|-----------|----------------|
| kube-scheduler | Selects the best node for a Pod |
| kube-apiserver | Stores scheduling decisions |
| kubelet | Creates and runs the Pod on the selected node |

---

# Workflow Diagram

```text
                 New Pod
                    │
                    ▼
             kube-apiserver
                    │
                    ▼
            kube-scheduler
                    │
      ┌─────────────┴─────────────┐
      │                           │
      ▼                           ▼
 Filter Nodes              Score Nodes
      │                           │
      └─────────────┬─────────────┘
                    ▼
          Select Best Node
                    │
                    ▼
            kube-apiserver
                    │
                    ▼
               kubelet
                    │
                    ▼
             Pod is Running
```

## Key Points

- The scheduler **only selects a node** for the Pod.
- The **kubelet** is responsible for creating and starting the Pod.
- Scheduling happens in **two phases**:
  1. **Filtering** – Remove unsuitable nodes.
  2. **Scoring** – Rank the remaining nodes and choose the best one.
- The scheduler makes placement decisions based on resource availability and scheduling policies.
```
