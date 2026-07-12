# Kube-apiserver

**Primary management component in Kubernetes**

When you run a `kubectl` command, the `kubectl` utility is in fact reaching to the kube API server.

The kube API server first:
- Authenticates the request
- Validates the request
- Retrieves the data from the `etcd` cluster
- Responds back with the requested information

## Example: Creating a Pod

When you create a Pod:

1. The request is authenticated first.
2. The request is validated.
3. The API server creates a Pod object without assigning it to a node.
4. The API server updates the information in the `etcd` server.
5. The API server updates the user that the Pod has been created.

The scheduler continuously monitors the kube-apiserver and realizes that there is a new Pod with no node assigned.

The scheduler:
- Identifies the right node to place the new Pod on.
- Communicates that back to the kube-apiserver.

The API server then:
1. Updates the information in the `etcd` cluster.
2. Passes that information to the kubelet in the appropriate worker node.

The kubelet then:
- Creates the Pod on the node.
- Instructs the container runtime engine to deploy the application image.

Once done:
1. The kubelet updates the status back to the API server.
2. The API server updates the data back in the `etcd` cluster.
