# Deployment

## What is a Deployment?

A **Deployment** is a Kubernetes workload object used to run and update a set of identical application Pods.

A Deployment does not normally create Pods directly. It manages a **ReplicaSet**, and the ReplicaSet creates and maintains the Pods.

```text
Deployment
   └── ReplicaSet
         └── Pods
```

<img width="1257" height="762" alt="deployment-architecture" src="https://github.com/user-attachments/assets/43230b43-a1de-427a-add4-0433ffbadf3a" />


A Deployment is the standard controller for **stateless applications**, such as web servers, APIs, and frontend services.

---

## Why use a Deployment?

A ReplicaSet can maintain a desired number of Pods, but a Deployment adds application lifecycle and release-management capabilities:

- Declarative application updates
- Rolling updates with controlled Pod replacement
- Rollback to an earlier revision
- Rollout history and status tracking
- Pause and resume during a rollout
- Horizontal scaling
- Automatic creation and management of ReplicaSets

The important point is:

> A ReplicaSet maintains Pod availability. A Deployment manages application releases through ReplicaSets.

---

## Deployment manifest

The following manifest creates a Deployment with three NGINX Pods:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
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
          ports:
            - containerPort: 80
```

<img width="668" height="712" alt="deployment-manifest" src="https://github.com/user-attachments/assets/efb78945-b1bd-4a0e-8396-f7aa7449c867" />

### Main fields

| Field | Purpose |
|---|---|
| `apiVersion: apps/v1` | Uses the stable Kubernetes applications API. |
| `kind: Deployment` | Declares a Deployment object. |
| `metadata.name` | Sets the Deployment name. |
| `spec.replicas` | Defines the desired number of Pods. |
| `spec.selector` | Identifies the Pods managed by the Deployment. |
| `spec.template` | Defines the Pod specification used to create new Pods. |
| `spec.template.spec.containers` | Defines the containers running inside each Pod. |

### Selector rule

The Deployment selector must match the labels in the Pod template:

```yaml
selector:
  matchLabels:
    app: myapp
    type: front-end

template:
  metadata:
    labels:
      app: myapp
      type: front-end
```

A mismatch is rejected by the Kubernetes API. In `apps/v1`, a Deployment's selector is also immutable after creation, so choose selectors carefully.

---

## Creating and inspecting the Deployment

Save the manifest as `deployment.yaml`, then run:

```bash
kubectl apply -f deployment.yaml
```

Inspect the resources:

```bash
kubectl get deployments
kubectl get replicasets
kubectl get pods
kubectl get all
```

Typical ownership chain:

```text
myapp-deployment
└── myapp-deployment-<pod-template-hash>
    ├── myapp-deployment-<pod-template-hash>-<pod-id-1>
    ├── myapp-deployment-<pod-template-hash>-<pod-id-2>
    └── myapp-deployment-<pod-template-hash>-<pod-id-3>
```

<img width="765" height="231" alt="kubectl-get-all-output" src="https://github.com/user-attachments/assets/7953de53-b359-4da5-a8ee-6ced255d304e" />


The generated hash distinguishes ReplicaSet revisions. During an update, the Deployment can create a new ReplicaSet with a different hash while scaling down the old one.

> `kubectl get all` displays many common workload and service resources, but it does not literally display every resource type in the namespace.

---

## Deployment vs ReplicaSet

| Capability | Deployment | ReplicaSet |
|---|---|---|
| Primary responsibility | Manages application rollout and lifecycle | Maintains a stable number of matching Pods |
| Manages | ReplicaSets and indirectly Pods | Pods directly |
| Pod self-healing | Yes, through its ReplicaSet | Yes |
| Scaling | Yes | Yes |
| Rolling updates | Yes | No built-in release orchestration |
| Rollback | Yes | No revision-based rollback |
| Rollout history | Yes | No |
| Pause and resume rollout | Yes | No |
| Typical user interaction | Create and manage directly | Usually created automatically by a Deployment |
| Recommended for stateless applications | Yes | Usually no, unless a Deployment is unnecessary |

### Major architectural difference

```text
ReplicaSet:
ReplicaSet ──controls──> Pods

Deployment:
Deployment ──controls──> ReplicaSet ──controls──> Pods
```

A ReplicaSet only reconciles the desired Pod count. It does not understand application versions as a release workflow.

A Deployment tracks Pod-template revisions by managing multiple ReplicaSets. This enables controlled migration from one application version to another.

---

## Rolling update example

Update the container image:

```bash
kubectl set image deployment/myapp-deployment \
  nginx-container=nginx:1.28
```

Watch the rollout:

```bash
kubectl rollout status deployment/myapp-deployment
kubectl get replicasets
kubectl get pods
```

During the rolling update, Kubernetes normally:

1. Creates a new ReplicaSet for the updated Pod template.
2. Gradually scales up the new ReplicaSet.
3. Gradually scales down the previous ReplicaSet.
4. Keeps the old ReplicaSet at zero replicas for possible rollback, subject to revision-history retention.

A rolling update can be controlled with:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

- `maxUnavailable` controls how many desired Pods may be unavailable during the update.
- `maxSurge` controls how many extra Pods may be created above the desired replica count.

---

## Rollback, pause, and resume

View rollout history:

```bash
kubectl rollout history deployment/myapp-deployment
```

Undo the latest rollout:

```bash
kubectl rollout undo deployment/myapp-deployment
```

Pause a rollout before applying several changes:

```bash
kubectl rollout pause deployment/myapp-deployment
```

Apply changes while paused, then continue:

```bash
kubectl rollout resume deployment/myapp-deployment
```

Pausing is useful when several Pod-template changes should be grouped into one rollout rather than producing multiple intermediate revisions.

---

## Scaling

Scale imperatively:

```bash
kubectl scale deployment myapp-deployment --replicas=5
```

Or update the manifest:

```yaml
spec:
  replicas: 5
```

Then apply it:

```bash
kubectl apply -f deployment.yaml
```

---

## Operational guidance

- Manage the **Deployment**, not the ReplicaSets created by it.
- Do not manually scale or edit an owned ReplicaSet unless performing controlled troubleshooting; the Deployment controller can overwrite the change.
- Use readiness probes so rolling updates only treat application-ready Pods as available.
- Use explicit image tags instead of mutable tags such as `latest`.
- Check `kubectl rollout status` before considering a release successful.
- Use a `Service` to provide a stable network endpoint for the Pods.
- Use a `StatefulSet` instead when Pods require stable identity or persistent per-Pod storage semantics.

---

## Quick summary

```text
Pod        = runs one or more containers
ReplicaSet = keeps the required number of Pods running
Deployment = manages ReplicaSets, versions, updates, and rollbacks
```

For normal stateless application workloads, create a **Deployment** rather than creating a ReplicaSet directly.

## References

- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Update a Deployment Without Downtime](https://kubernetes.io/docs/tasks/run-application/update-deployment-rolling/)
