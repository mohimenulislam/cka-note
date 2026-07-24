# Kubernetes Service — LoadBalancer

A **LoadBalancer Service** exposes an application outside the Kubernetes cluster through an external load balancer.

<img width="1438" height="818" alt="01-voting-app-architecture" src="https://github.com/user-attachments/assets/c02ae090-a509-46b1-bcd4-6f063f83a5df" />

## Voting Application Example

The application contains these components:

- `voting-app`
- `result-app`
- `redis`
- `db`
- `worker`

The `voting-app` and `result-app` must be accessible by users.

Internal components such as Redis and the database are connected through internal Kubernetes Services.

## Problem with NodePort

With a NodePort Service, users access the application by using a node IP and a high port number.

Example:

```text
http://192.168.56.70:30035
http://192.168.56.71:30035
http://192.168.56.72:30035
http://192.168.56.73:30035
```

For the result application:

```text
http://192.168.56.70:31061
http://192.168.56.71:31061
http://192.168.56.72:31061
http://192.168.56.73:31061
```

<img width="1466" height="766" alt="02-nodeport-access" src="https://github.com/user-attachments/assets/a9291dea-1a73-47f4-bc7a-8d0938f97211" />

This is not convenient for end users because they must know:

- a node IP address;
- the NodePort number;
- which URL belongs to which application.

## Desired Access

Users normally expect one simple URL for each application:

```text
http://example-vote.com
http://example-result.com
```

<img width="1496" height="772" alt="03-single-url-goal" src="https://github.com/user-attachments/assets/b50ab918-6900-49ba-a34a-9be99701205c" />

## LoadBalancer Service

A LoadBalancer Service provides a single external entry point.

```text
User
  |
  v
External Load Balancer
  |
  v
Kubernetes Service
  |
  v
Application Pods
```

The external load balancer forwards requests to the Kubernetes cluster, and the Service sends traffic to the matching Pods.

<img width="1493" height="765" alt="04-loadbalancer-flow" src="https://github.com/user-attachments/assets/8ef274e4-0caa-477c-be1c-dfe62c37aad4" />

## LoadBalancer Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: LoadBalancer

  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
```
<img width="1502" height="772" alt="05-loadbalancer-yaml" src="https://github.com/user-attachments/assets/4b47dbb6-5fcc-4283-b6bf-9d10399abb26" />

## Port Meaning

| Field | Meaning |
|---|---|
| `targetPort` | Port used by the application inside the Pod |
| `port` | Port exposed by the Kubernetes Service |
| `nodePort` | Port opened on the cluster nodes |

Traffic flow:

```text
External Load Balancer
        |
        v
Service port: 80
        |
        v
Pod targetPort: 80
```

## Selector

A real Service must select the application Pods using labels.

Example:

```yaml
selector:
  app: myapp
```

The selector must match the Pod labels:

```yaml
labels:
  app: myapp
```

## Complete Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: LoadBalancer

  selector:
    app: myapp

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
```

## Create and Check the Service

Create the Service:

```bash
kubectl create -f service-definition.yml
```

Check the Service:

```bash
kubectl get services
```

Example output:

```text
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)
myapp-service   LoadBalancer   10.96.120.10    <external-ip>    80:30008/TCP
```

## Important Point

A LoadBalancer Service depends on an environment that can provide an external load balancer.

In a supported cloud environment, Kubernetes requests a cloud load balancer.

In a local environment without a load-balancer implementation, the Service can remain without an external IP.

## Summary

```text
NodePort
  = Node IP + high port

LoadBalancer
  = Single external entry point
```
