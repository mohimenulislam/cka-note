# Kubernetes YAML for Pods

## What is YAML?

YAML (**YAML Ain't Markup Language**) is a human-readable data serialization language used by Kubernetes to define objects such as:

- Pod
- Service
- ReplicaSet
- Deployment
- DaemonSet
- StatefulSet
- ConfigMap
- Secret

Instead of creating resources manually using CLI commands, Kubernetes reads a YAML file and creates the desired object.

---

# Kubernetes YAML Structure

Every Kubernetes YAML file contains **four top-level fields**.

```yaml
apiVersion:
kind:
metadata:
spec:
```

> **These four fields are mandatory for almost every Kubernetes resource.**

---

# 1. apiVersion

The `apiVersion` specifies **which Kubernetes API version** should be used to create the object.

Example:

```yaml
apiVersion: v1
```

### Common API Versions

| Resource | API Version |
|-----------|-------------|
| Pod | v1 |
| Service | v1 |
| ConfigMap | v1 |
| Secret | v1 |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |
| DaemonSet | apps/v1 |
| StatefulSet | apps/v1 |

> **Always use the correct API version for the resource type.**

---

# 2. kind

The `kind` field defines **what Kubernetes object** you want to create.

Example

```yaml
kind: Pod
```

Other examples

```yaml
kind: Service
kind: ReplicaSet
kind: Deployment
```

---

# 3. metadata

The `metadata` section stores information **about the object**.

Example

```yaml
metadata:
  name: myapp-pod
```

Metadata commonly contains:

- name
- labels
- annotations
- namespace

---

## metadata Example

```yaml
metadata:
  name: myapp-pod

  labels:
    app: myapp
```

---

# Labels

Labels are **key-value pairs** used to organize Kubernetes resources.

Example

```yaml
labels:
  app: myapp
```

Multiple labels

```yaml
labels:
  app: myapp
  type: front-end
```

---

## Why Labels?

Labels are used to

- Group Pods
- Filter Pods
- Connect Services to Pods
- Select Pods for ReplicaSets
- Select Pods for Deployments

Example

```bash
kubectl get pods --show-labels
```

Filter Pods

```bash
kubectl get pods -l app=myapp
```

Multiple labels

```bash
kubectl get pods -l app=myapp,type=front-end
```

---

# YAML Indentation

YAML is **indentation-sensitive**.

✅ Correct

```yaml
metadata:
  name: myapp-pod

  labels:
    app: myapp
```

❌ Incorrect

```yaml
metadata:
    name: myapp-pod
      labels:
    app: myapp
```

> **Incorrect indentation will cause Kubernetes to reject the YAML file.**

---

# 4. spec

The `spec` section defines **how the object should behave**.

Example

```yaml
spec:
```

Each Kubernetes object has its own specification.

Examples

- Pod spec
- Deployment spec
- Service spec
- ReplicaSet spec

---

# Pod Spec

For Pods, one important property is:

```yaml
containers:
```

Example

```yaml
spec:
  containers:
```

---

# containers

`containers` is a **List (Array)**.

Why?

A Pod can contain **one or more containers**.

Example

```yaml
containers:
```

means

```
Container 1
Container 2
Container 3
```

---

# Single Container Pod

Most Pods contain only one container.

Example

```yaml
containers:
  - name: nginx-container
    image: nginx
```

Notice the dash (`-`).

```yaml
-
```

The dash represents the **first item in the list**.

Example with multiple containers

```yaml
containers:
  - name: nginx

  - name: redis

  - name: sidecar
```

---

# Container Properties

Each container contains properties such as

```yaml
- name: nginx-container
  image: nginx
```

| Property | Description |
|----------|-------------|
| name | Container name |
| image | Docker image |

---

# Complete Pod YAML

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: myapp-pod

  labels:
    app: myapp
    type: front-end

spec:
  containers:
    - name: nginx-container
      image: nginx
```

---

# YAML Hierarchy

```
apiVersion
kind

metadata
├── name
└── labels
    ├── app
    └── type

spec
└── containers
    └── List
        ├── name
        └── image
```

---

# Creating the Pod

Save the file

```
pod-definition.yaml
```

Create the Pod

```bash
kubectl create -f pod-definition.yaml
```

or

```bash
kubectl apply -f pod-definition.yaml
```

---

# Verify the Pod

```bash
kubectl get pods
```

Example

```
NAME        READY   STATUS
myapp-pod   1/1     Running
```

---

# Describe the Pod

```bash
kubectl describe pod myapp-pod
```

Displays:

- Pod Name
- Namespace
- Labels
- Node
- IP Address
- Container Image
- Status
- Events
- Conditions
- Mounted Volumes

---

# Common Commands

### Create a Pod

```bash
kubectl create -f pod-definition.yaml
```

---

### Apply Changes

```bash
kubectl apply -f pod-definition.yaml
```

---

### List Pods

```bash
kubectl get pods
```

---

### Show Labels

```bash
kubectl get pods --show-labels
```

---

### Describe Pod

```bash
kubectl describe pod myapp-pod
```

---

### Delete Pod

```bash
kubectl delete pod myapp-pod
```

---

### View Running Pod YAML

```bash
kubectl get pod myapp-pod -o yaml
```

---

# YAML Syntax Rules

✅ Use spaces (not tabs)

✅ Maintain proper indentation

✅ Keys end with `:`

✅ Lists begin with `-`

✅ Dictionaries are created using indentation

---

# Important Notes

> **Every Kubernetes YAML file starts with four mandatory fields:**
>
> - `apiVersion`
> - `kind`
> - `metadata`
> - `spec`

---

> **`metadata` contains information about the object such as name, labels, annotations, and namespace.**

---

> **Labels are key-value pairs used for organizing, filtering, and selecting Kubernetes resources.**

---

> **The `spec` section defines the desired state of the Kubernetes object.**

---

> **`containers` is always a list because a Pod can contain one or more containers.**

---

> **The dash (`-`) indicates an item in a YAML list.**

---

> **YAML is indentation-sensitive. Incorrect indentation causes parsing errors.**

---

> **Use `kubectl create -f` to create a new resource and `kubectl apply -f` to create or update an existing resource.**

---
