# Kubelet

The **kubelet** is an agent that runs on **every worker node** in a Kubernetes cluster.

It is responsible for ensuring that containers described in Pod specifications are running and healthy.

> **Important:** The kubelet does **not decide** where a Pod should run. That decision is made by the **kube-scheduler**. The kubelet simply receives instructions and executes them on its node.

---

# Responsibilities of kubelet

The kubelet performs several important tasks:

- Registers the worker node with the Kubernetes cluster.
- Receives Pod specifications from the API Server.
- Starts Pods using the container runtime.
- Monitors Pods and containers continuously.
- Reports node and Pod status back to the API Server.
- Restarts failed containers when necessary.

---

# How kubelet Works

When a Pod is scheduled to a worker node:

1. The scheduler selects the best node.
2. The API Server stores the scheduling decision.
3. The kubelet on that worker node detects the new Pod.
4. The kubelet requests the container runtime to pull the required image.
5. The container runtime creates and starts the containers.
6. The kubelet continuously monitors the Pod and reports its status to the API Server.

---

# kubelet Workflow

```text
           User
             │
             ▼
      kubectl apply
             │
             ▼
      kube-apiserver
             │
             ▼
      kube-scheduler
             │
             ▼
      Worker Node Selected
             │
             ▼
          kubelet
             │
             ▼
     Container Runtime
(containerd / CRI-O / Docker)
             │
             ▼
        Pull Image
             │
             ▼
        Create Pod
             │
             ▼
      Monitor Pod Health
             │
             ▼
      Update API Server
```

---

# Node Registration

When the kubelet starts, it registers the worker node with the Kubernetes cluster.

```text
Worker Node
     │
     ▼
kubelet starts
     │
     ▼
Registers Node
     │
     ▼
API Server
```

After registration, the node becomes visible:

```bash
kubectl get nodes
```

Example:

```text
NAME          STATUS   ROLES    AGE
worker-01     Ready    <none>   20d
worker-02     Ready    <none>   20d
```

---

# Pod Creation Process

Suppose a Pod is assigned to **worker-01**.

The kubelet performs the following steps:

```
API Server
      │
      ▼
kubelet detects Pod
      │
      ▼
Container Runtime
      │
      ▼
Pull Image
      │
      ▼
Create Containers
      │
      ▼
Start Pod
```

---

# Pod Monitoring

After the Pod starts, the kubelet continuously monitors:

- Pod status
- Container health
- Restart count
- Resource usage

If a container crashes:

```
Container Crash
        │
        ▼
kubelet detects failure
        │
        ▼
Restart Container
```

---

# Communication with Container Runtime

The kubelet communicates with a **Container Runtime Interface (CRI)** implementation.

Supported runtimes include:

- containerd
- CRI-O
- Docker (via cri-dockerd)

The kubelet instructs the runtime to:

- Pull container images
- Create containers
- Start containers
- Stop containers
- Delete containers

---

# kubelet Installation

Unlike control plane components, the kubelet must be installed **manually** on every worker node.

Typical steps:

1. Install the kubelet package.
2. Configure the kubelet.
3. Run it as a system service.
4. Join the node to the cluster using `kubeadm join`.

---

# Viewing kubelet Process

Check whether the kubelet is running:

```bash
ps -aux | grep kubelet
```

Example output:

```text
root   1823   ... kubelet
```

---

# Check kubelet Service

```bash
systemctl status kubelet
```

Start kubelet:

```bash
sudo systemctl start kubelet
```

Enable kubelet at boot:

```bash
sudo systemctl enable kubelet
```

---

# kubelet Configuration

Common kubelet configuration locations:

```text
/var/lib/kubelet/config.yaml
```

Systemd service:

```text
/etc/systemd/system/kubelet.service
```

---

# kubelet vs kube-scheduler

| Component | Responsibility |
|-----------|----------------|
| kube-scheduler | Selects the best node for a Pod |
| kubelet | Creates and manages Pods on its node |
| kube-apiserver | Stores cluster state and coordinates communication |
| Container Runtime | Runs the actual containers |

---

# Overall Workflow

```text
             kubectl
                 │
                 ▼
         kube-apiserver
                 │
                 ▼
        kube-scheduler
                 │
                 ▼
        Select Worker Node
                 │
                 ▼
             kubelet
                 │
                 ▼
      Container Runtime
                 │
                 ▼
           Pull Image
                 │
                 ▼
            Create Pod
                 │
                 ▼
        Monitor Pod Health
                 │
                 ▼
      Update API Server
```

---

# Key Points

- The **kubelet runs on every worker node**.
- It **registers the node** with the Kubernetes cluster.
- It receives Pod assignments from the **API Server**.
- It communicates with the **container runtime** to create and manage containers.
- It continuously monitors Pods and reports their status to the **API Server**.
- The kubelet **does not schedule Pods**; scheduling is handled by the **kube-scheduler**.
- Unlike most control plane components, the kubelet is **installed and managed as a system service** on each node.
