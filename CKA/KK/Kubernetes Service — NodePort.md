# Kubernetes Service — NodePort

A Kubernetes **Service** is an API object that provides a stable network endpoint for a changing set of Pods. Pods can be recreated, rescheduled, and assigned new IP addresses; clients should therefore connect to the Service instead of depending on individual Pod IPs.

> A Service is often described as a “virtual server,” but it is not a virtual machine or a standalone server process. It is a virtual networking abstraction implemented by the cluster data plane, commonly through `kube-proxy` or an eBPF-based networking implementation.


<img width="1160" height="696" alt="01-service-overview" src="https://github.com/user-attachments/assets/b26f5274-0abd-48d7-806d-4e3e4623ed96" />


## Why a Service is required

Each Pod receives an IP address from the cluster Pod network. That IP is normally routable inside the cluster, but it is usually not directly reachable from a workstation outside the cluster.

The screenshots use this example:

- Workstation IP: `192.168.1.10`
- Kubernetes node IP: `192.168.1.2`
- Pod network: `10.244.0.0/16`
- Pod IP: `10.244.0.2`

A request from inside the node can reach the Pod directly:

```bash
curl http://10.244.0.2
```


<img width="1224" height="424" alt="02-pod-access-from-node" src="https://github.com/user-attachments/assets/f2d9e219-d43c-4501-8779-1e1f999d5922" />


An external workstation usually cannot use the Pod IP directly because the Pod network is private to the cluster.



<img width="1213" height="435" alt="03-external-client-cannot-reach-pod-ip" src="https://github.com/user-attachments/assets/b4e900ba-228c-4eac-96b2-472606ec9a9e" />


A `NodePort` Service exposes the application through a port on the node:

```text
http://<NodeIP>:<NodePort>
```

Example:

```bash
curl http://192.168.1.2:30008
```


<img width="1214" height="388" alt="04-nodeport-service-external-access" src="https://github.com/user-attachments/assets/36dc9a54-996c-4562-a645-ab51ac584877" />


## Service types

<img width="1391" height="580" alt="05-service-types" src="https://github.com/user-attachments/assets/d93ad270-0daf-49ea-a3ca-ef2eff7e5c3b" />


| Service type | Reachability | Typical purpose |
|---|---|---|
| `ClusterIP` | Inside the cluster | Default type for service-to-service communication |
| `NodePort` | `<NodeIP>:<NodePort>` | Simple external access, labs, testing, or as a building block for other exposure methods |
| `LoadBalancer` | External load-balancer address | Cloud or integrated load-balancer exposure |
| `ExternalName` | DNS CNAME | Maps a Service name to an external DNS name |

A `NodePort` Service also receives a **ClusterIP**. Traffic can therefore reach it through either:

```text
<ClusterIP>:<ServicePort>      # from inside the cluster
<NodeIP>:<NodePort>            # from outside, if the node is reachable
```

## NodePort traffic flow

```text
Client
  |
  |  http://<NodeIP>:30008
  v
NodePort 30008 on a cluster node
  |
  v
Service ClusterIP:80
  |
  v
Selected PodIP:80
```


<img width="1273" height="601" alt="06-nodeport-traffic-flow" src="https://github.com/user-attachments/assets/f4550b52-ee27-4248-aa39-dbe064e3869e" />


## The three ports

A NodePort Service commonly involves three different port values.

<img width="966" height="619" alt="09-nodeport-range" src="https://github.com/user-attachments/assets/79f50e89-b2ec-480f-9a89-12928ec73bd7" />


| Field | Example | Meaning |
|---|---:|---|
| `targetPort` | `80` | Destination port on the selected Pod/container |
| `port` | `80` | Port exposed by the Service on its ClusterIP |
| `nodePort` | `30008` | Port exposed on each eligible node IP |

### 1. `targetPort`

`targetPort` is where the application is listening in the selected Pods.

```yaml
ports:
  - targetPort: 80
```

<img width="1448" height="652" alt="11-targetport-field" src="https://github.com/user-attachments/assets/87f71eb1-dfad-4369-950f-87f70f11914e" />

A named target port is usually clearer:

```yaml
# Deployment container
ports:
  - name: http
    containerPort: 80

# Service
ports:
  - port: 80
    targetPort: http
```

### 2. `port`

`port` is the port exposed by the Service itself on the Service ClusterIP.

```yaml
ports:
  - port: 80
```

<img width="1429" height="601" alt="12-service-port-field" src="https://github.com/user-attachments/assets/98a86bae-8769-4ff7-b502-3c7bc17dbf42" />

Inside the cluster, a client can use:

```text
http://myapp-service:80
```

or the Service ClusterIP:

```text
http://<ClusterIP>:80
```

### 3. `nodePort`

`nodePort` is the externally reachable port opened for the Service on the nodes.

```yaml
ports:
  - nodePort: 30008
```

<img width="1421" height="608" alt="13-nodeport-field" src="https://github.com/user-attachments/assets/d0d3e38e-a76f-48ea-888a-61affc6f27c8" />

The conventional default allocation range is:

```text
30000-32767
```

The actual range is cluster-configurable. When `nodePort` is omitted, Kubernetes normally allocates an available port from the configured range.

## Complete NodePort Service manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
    tier: frontend
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: http
      nodePort: 30008
```

A matching Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      tier: frontend
  template:
    metadata:
      labels:
        app: myapp
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.27-alpine
          ports:
            - name: http
              containerPort: 80
```

The same manifests are included in this repository:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      tier: frontend
  template:
    metadata:
      labels:
        app: myapp
        tier: frontend
    spec:
      containers:
        - name: nginx
          image: nginx:1.27-alpine
          ports:
            - name: http
              containerPort: 80

```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
    tier: frontend
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: http
      nodePort: 30008

```

## How the Service finds Pods

A Service does not select Pods by name or IP address. It selects them through labels.

Service selector:

```yaml
selector:
  app: myapp
  tier: frontend
```

Pod template labels:

```yaml
labels:
  app: myapp
  tier: frontend
```

The selector must match the Pod labels exactly for every selector key.

<img width="1290" height="701" alt="15-service-selector-matches-pod-labels" src="https://github.com/user-attachments/assets/f8eb95d1-5763-4f34-b5d3-c271a985e5e7" />


Kubernetes creates and updates EndpointSlice objects containing the ready backend endpoints. When Pods are added, deleted, or replaced, the Service endpoint set is updated automatically.

## Create and verify

Apply both resources:

```bash
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service-nodeport.yaml
```

Inspect the Service:

```bash
kubectl get services
kubectl get service myapp-service -o wide
kubectl describe service myapp-service
```

Inspect the selected endpoints:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=myapp-service
```

Expected Service output resembles:

```text
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
myapp-service   NodePort   10.106.127.123  <none>        80:30008/TCP   1m
```

In the `PORT(S)` column:

```text
80:30008/TCP
|  |     |
|  |     +-- protocol
|  +-------- nodePort
+----------- Service port
```


<img width="775" height="293" alt="16-kubectl-create-and-get-services" src="https://github.com/user-attachments/assets/66b89dbe-db07-4e55-8138-505ec12bbf42" />


Test from a machine that can reach the node IP:

```bash
curl http://192.168.1.2:30008
```


<img width="815" height="597" alt="17-curl-nodeport-test" src="https://github.com/user-attachments/assets/2e4bb6de-9992-44b7-ad2d-76cc80a0ef16" />


## Multiple Pods

When several ready Pods have labels matching the Service selector, they all become Service endpoints.


<img width="777" height="575" alt="18-nodeport-multiple-pods" src="https://github.com/user-attachments/assets/7663cee9-7a10-44d5-b4af-6424f657bc4d" />

The Service provides one stable endpoint while the backend Pod set changes dynamically:

```text
<NodeIP>:30008
      |
      v
myapp-service
  |    |    |
Pod1 Pod2 Pod3
```

Do not rely on a specific portable “random load-balancing algorithm.” Endpoint selection depends on the Service proxy/data-plane implementation, proxy mode, connection tracking, traffic policies, and session-affinity settings. The correct operational assumption is that new connections can be routed to any eligible ready endpoint unless policy or affinity changes that behavior.

## NodePort across multiple nodes

For a NodePort Service, the same NodePort is normally exposed on each eligible node. With the default external traffic policy, traffic that enters one node can be forwarded to a selected ready Pod on another node.

<img width="1295" height="594" alt="21-nodeport-across-multiple-nodes" src="https://github.com/user-attachments/assets/d34c34a5-97e1-472f-8527-fa08ccd11da3" />

Examples:

```bash
curl http://192.168.1.2:30008
curl http://192.168.1.3:30008
curl http://192.168.1.4:30008
```

All requests address the same Kubernetes Service, provided the network and firewall permit access to that NodePort on those node IPs.

### `externalTrafficPolicy`

The default is:

```yaml
spec:
  externalTrafficPolicy: Cluster
```

This allows a node receiving traffic to forward it to ready endpoints across the cluster.

Using:

```yaml
spec:
  externalTrafficPolicy: Local
```

restricts external traffic to node-local ready endpoints and can preserve the client source IP, but a node without a local ready endpoint will not forward the request for that Service.

## Important operational notes

1. **NodePort does not create a public IP.** The node IP must already be reachable from the client.
2. **Firewalls and security groups must allow the NodePort.** For this example, permit TCP `30008` only from required source networks.
3. **The Service selector must match Pod labels.** A selector mismatch creates a Service with no useful endpoints.
4. **`targetPort` must match the application listener.** A correct selector with a wrong target port still fails.
5. **Readiness affects endpoints.** Pods that are not ready are normally excluded from Service traffic.
6. **Pod IPs are not stable API endpoints.** Use Service DNS or the Service ClusterIP inside the cluster.
7. **For production HTTP/HTTPS, NodePort is usually not the final architecture.** Prefer a supported `LoadBalancer`, Gateway API, or Ingress implementation for managed external access, TLS, and routing.
8. **Avoid hard-coding a NodePort unless required.** Automatic allocation reduces collision risk.

## Troubleshooting

### Service has no endpoints

```bash
kubectl get pods --show-labels
kubectl get service myapp-service -o yaml
kubectl get endpointslice \
  -l kubernetes.io/service-name=myapp-service
```

Check:

- Selector keys and values
- Pod readiness
- Namespace alignment
- Deployment replica count

### NodePort times out

Check:

```bash
kubectl get nodes -o wide
kubectl describe service myapp-service
kubectl get pods -o wide
```

Then verify:

- The selected node IP is reachable
- TCP `30008` is allowed through host firewall/security groups
- The Service has ready endpoints
- The container listens on `targetPort`
- NetworkPolicy permits the traffic path
- The cluster networking implementation supports the intended NodePort path

### Test from inside the cluster

```bash
kubectl run curl-test \
  --image=curlimages/curl:8.10.1 \
  --restart=Never \
  --rm -it -- \
  curl http://myapp-service:80
```

If the internal Service test works but `<NodeIP>:<NodePort>` fails, investigate node reachability, firewall rules, proxy/data-plane configuration, and external traffic policy.

## Summary

```text
Service object
├── Stable ClusterIP
├── Stable DNS name
├── Label selector
├── Dynamic EndpointSlices
└── Optional external exposure
    └── NodePort: <NodeIP>:<NodePort>
```

For the example in these notes:

```text
Client request:   192.168.1.2:30008
NodePort:         30008
Service port:     80
Service ClusterIP: assigned by Kubernetes
Pod targetPort:   80
Backend Pods:     selected using app=myapp,tier=frontend
```

## Screenshot appendix

The `images/` directory contains all screenshots extracted from the supplied document, including diagrams, YAML walkthroughs, command output, multi-Pod routing, multi-node behavior, and lecture transcript captures.

## Official references

- [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Virtual IPs and Service Proxies](https://kubernetes.io/docs/reference/networking/virtual-ips/)
- [Services, Load Balancing, and Networking](https://kubernetes.io/docs/concepts/services-networking/)
