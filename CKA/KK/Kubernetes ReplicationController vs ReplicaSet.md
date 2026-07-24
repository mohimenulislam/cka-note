# Kubernetes ReplicationController vs ReplicaSet

> Quick GitHub notes on Kubernetes pod replication, based on the supplied YAML diagrams.

## Core idea

Both **ReplicationController (RC)** and **ReplicaSet (RS)** are reconciliation controllers. They continuously compare:

- **Desired state** — `.spec.replicas`
- **Current state** — Pods matching `.spec.selector`

If too few matching Pods exist, the controller creates more from `.spec.template`. If too many exist, it removes the excess.

<img width="664" height="648" alt="replication-controller-structure" src="https://github.com/user-attachments/assets/7eca535d-5d8b-42b5-9559-0091377d9cd7" />


---

## 1. ReplicationController

A **ReplicationController** maintains a requested number of identical Pods.

- API: `v1`
- Kind: `ReplicationController`
- Short name: `rc`
- Selector type: equality-based key/value map only
- Status: **legacy**; replaced by ReplicaSet and Deployment for modern workloads

### Example

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    app: myapp
    type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.27
```

### Notes

- `spec.template` is the Pod template nested inside the controller.
- `replicas`, `selector`, and `template` are siblings under the controller's top-level `spec`.
- When an RC selector is omitted, Kubernetes can default it from the Pod template labels.
- RC supports only simple equality-based selectors such as `app: myapp`.


<img width="672" height="645" alt="replication-controller-replicas" src="https://github.com/user-attachments/assets/55c00a8f-beee-4898-9a36-0d687dc494f3" />

---

## 2. ReplicaSet

A **ReplicaSet** also maintains a stable number of matching Pods, but it uses the newer `apps/v1` workload API and a richer selector structure.

- API: `apps/v1`
- Kind: `ReplicaSet`
- Short name: `rs`
- Selector types: `matchLabels` and `matchExpressions`
- Normal usage: created and managed by a **Deployment**

### Example

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.27
```

The ReplicaSet selector is required and must match the labels in `spec.template.metadata.labels`.

<img width="710" height="707" alt="replicaset-structure" src="https://github.com/user-attachments/assets/74c3e848-a132-4e79-8ac1-6d34bb8eedcd" />


### Set-based selector example

ReplicaSet can use more expressive selection logic:

```yaml
selector:
  matchLabels:
    app: myapp
  matchExpressions:
    - key: environment
      operator: In
      values:
        - production
        - staging
```

All selector requirements are combined using logical **AND**.

---

## Major changes: ReplicationController → ReplicaSet

| Area | ReplicationController | ReplicaSet |
|---|---|---|
| API version | `v1` | `apps/v1` |
| Resource kind | `ReplicationController` | `ReplicaSet` |
| Operational status | Legacy | Current low-level replication controller |
| Selector syntax | Flat equality map | `matchLabels` plus `matchExpressions` |
| Set-based selectors | Not supported | Supported: `In`, `NotIn`, `Exists`, `DoesNotExist` |
| Selector requirement | Can default from template labels | Explicit selector is required |
| Template relationship | Selector should match template labels | Selector **must** match template labels |
| Recommended use | Maintain old manifests only | Usually managed indirectly through a Deployment |
| Rolling updates and rollback | Manual / limited | ReplicaSet alone does not provide rollout management; Deployment does |

## Most important practical difference

The main technical improvement is the **selector model**.

ReplicationController:

```yaml
selector:
  type: front-end
```

ReplicaSet:

```yaml
selector:
  matchLabels:
    type: front-end
```

A ReplicaSet can also add `matchExpressions`, which allows set-based matching. This makes ReplicaSet selectors more expressive and compatible with modern workload APIs.

## Selector safety

A controller can acquire existing unmanaged Pods whose labels match its selector. Therefore, this selector is too broad in a namespace containing multiple frontend applications:

```yaml
matchLabels:
  type: front-end
```

A safer selector uses enough labels to uniquely identify the workload:

```yaml
matchLabels:
  app: myapp
  type: front-end
```

Also avoid overlapping selectors across different ReplicaSets in the same namespace.

---

## Production recommendation

For application workloads, use a **Deployment** instead of creating a ReplicaSet directly:

```text
Deployment
   └── ReplicaSet
         └── Pods
```

A Deployment adds controlled rollouts, rollout history, rollback, and declarative Pod-template updates. The ReplicaSet remains the component that maintains the replica count.

---

## Useful commands

```bash
# Create resources
kubectl apply -f replication-controller.yaml
kubectl apply -f replicaset.yaml

# List controllers
kubectl get rc
kubectl get rs

# Inspect state
kubectl describe rc myapp-rc
kubectl describe rs myapp-replicaset

# Scale
kubectl scale rc myapp-rc --replicas=5
kubectl scale rs myapp-replicaset --replicas=5

# Show Pods selected by labels
kubectl get pods -l app=myapp,type=front-end
```

---

## Official references

- [Kubernetes: ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)
- [Kubernetes: ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Kubernetes API: ReplicationController v1](https://kubernetes.io/docs/reference/kubernetes-api/core/replication-controller-v1/)
- [Kubernetes API: ReplicaSet apps/v1](https://kubernetes.io/docs/reference/kubernetes-api/apps/replica-set-v1/)
- [Kubernetes: Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
