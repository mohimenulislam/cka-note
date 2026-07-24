# Kubernetes Service — ClusterIP

A **ClusterIP Service** provides one stable internal IP address and DNS name for a group of Pods.

Pods can be restarted, deleted, recreated, or scaled. Their individual Pod IP addresses may change. Applications should therefore connect to a Service instead of connecting directly to Pod IPs.

<img width="989" height="745" alt="clusterip-architecture" src="https://github.com/user-attachments/assets/96589ad1-d9bf-4e6f-be41-d6f5967a293f" />

## How ClusterIP Works

In the diagram:

- Front-end Pods communicate with the `back-end` Service.
- The `back-end` Service forwards traffic to matching backend Pods.
- Backend Pods communicate with the `redis` Service.
- The `redis` Service forwards traffic to matching Redis Pods.

```text
Front-end Pods
      |
      v
back-end ClusterIP Service
      |
      v
Back-end Pods
      |
      v
redis ClusterIP Service
      |
      v
Redis Pods
```

Each Service gives the application tier a single stable endpoint.

## Main Characteristics

- `ClusterIP` is the default Kubernetes Service type.
- It is used for communication inside the cluster.
- It receives a virtual IP called the **ClusterIP**.
- It selects Pods by matching labels.
- It forwards traffic to one of the selected Pods.
- It is not directly accessible from outside the cluster.

## ClusterIP Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: myapp
    type: back-end
```

### Field Explanation

| Field | Purpose |
|---|---|
| `apiVersion: v1` | API version used for a Service |
| `kind: Service` | Creates a Kubernetes Service object |
| `name: back-end` | Name of the Service |
| `type: ClusterIP` | Creates an internal-only Service |
| `port: 80` | Port exposed by the Service |
| `targetPort: 80` | Port used by the application inside the Pod |
| `selector` | Finds Pods with matching labels |

## Matching Pod Labels

The Service selector must match the labels defined on the Pods.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: back-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

The following values must match:

```yaml
# Service
selector:
  app: myapp
  type: back-end
```

```yaml
# Pod
labels:
  app: myapp
  type: back-end
```

<img width="1212" height="525" alt="clusterip-yaml" src="https://github.com/user-attachments/assets/65aa923a-67e3-4e3b-9f06-0a502c6554a9" />

## Create and Verify

Create the Pod and Service:

```bash
kubectl create -f pod-definition.yml
kubectl create -f service-definition.yml
```

Check the Service:

```bash
kubectl get services
```

Example output:

```text
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)
back-end   ClusterIP   10.96.10.25    <none>        80/TCP
```

## Access the Service

Another Pod in the same namespace can access the Service by name:

```bash
curl http://back-end
```

It can also use the ClusterIP:

```bash
curl http://10.96.10.25
```

Using the Service name is preferred because the application does not need to know the Service IP.

## Key Point

```text
ClusterIP Service = stable internal endpoint for a changing group of Pods
```
