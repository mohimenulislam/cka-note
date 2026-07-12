# Kube Controller Manager

The **Kube Controller Manager** is responsible for running various controllers in Kubernetes.

A **controller** is a process that continuously monitors the state of objects in the cluster and works towards bringing them to the desired state.

Think of a controller as an automated control loop that watches the current state of the cluster and takes corrective actions whenever necessary.

---

## How Controllers Work

A controller:

- Watches the current state of cluster resources.
- Compares the current state with the desired state.
- Takes necessary actions if there is any difference.
- Communicates with the cluster through the **Kube API Server**.

---

# Node Controller

The **Node Controller** is responsible for monitoring the health and status of worker nodes.

### Responsibilities

- Monitors the status of all nodes.
- Detects node failures.
- Marks unhealthy nodes.
- Reschedules workloads from failed nodes.

The Node Controller communicates with nodes through the **Kube API Server**.

### Node Monitoring

The Node Controller checks the status of every node **every 5 seconds**.

If it stops receiving heartbeat messages from a node:

- Waits **40 seconds**
- Marks the node as **Unreachable**

If the node still does not recover:

- Waits **5 minutes**
- Removes Pods running on that node
- Schedules those Pods on healthy nodes

---

# Replication Controller

The **Replication Controller** is responsible for ensuring that the desired number of Pods are always running.

### Responsibilities

- Monitors ReplicaSets
- Ensures the desired number of Pods are available
- Creates replacement Pods if an existing Pod fails

Example:

Desired Pods:

```
3 Pods
```

Current State:

```
Pod-1 ✅
Pod-2 ❌
Pod-3 ✅
```

Replication Controller creates:

```
Pod-4 ✅
```

Result:

```
3 Running Pods
```

---

# Other Kubernetes Controllers

Node Controller and Replication Controller are only two examples.

Kubernetes contains many controllers such as:

- Deployment Controller
- ReplicaSet Controller
- Namespace Controller
- Service Controller
- Endpoint Controller
- Persistent Volume Controller
- Job Controller
- CronJob Controller
- StatefulSet Controller
- DaemonSet Controller

Almost every Kubernetes object has its own controller.

---

# Where are Controllers Located?

All Kubernetes controllers run inside a single process called the:

```
kube-controller-manager
```

When Kubernetes is installed, all required controllers are packaged inside this single component.

---

# Viewing kube-controller-manager

The method depends on how Kubernetes was installed.

---

## kubeadm Installation

If the cluster was installed using **kubeadm**, the Controller Manager runs as a **Static Pod**.

Configuration file:

```bash
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

---

## Non-kubeadm Installation

If Kubernetes was installed manually or using another installer, the Controller Manager usually runs as a systemd service.

View its service configuration:

```bash
cat /etc/systemd/system/kube-controller-manager.service
```

---

## Check Running Process

You can verify whether the Controller Manager is running using:

```bash
ps -aux | grep kube-controller-manager
```

Example output:

```text
root   1234   ... kube-controller-manager
```

---

# Summary

| Component | Responsibility |
|-----------|----------------|
| Node Controller | Monitors node health and handles node failures |
| Replication Controller | Maintains desired number of Pods |
| kube-controller-manager | Runs all Kubernetes controllers |
| API Server | Controllers communicate with the cluster through the API Server |

---

## Workflow

```text
                +--------------------+
                | kube-apiserver     |
                +--------------------+
                          ▲
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
  Node Controller   Replica Controller   Deployment Controller
          │               │                │
          └───────────────┼────────────────┘
                          │
                          ▼
              kube-controller-manager
```
