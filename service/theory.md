# Kubernetes Services: ClusterIP, NodePort, and LoadBalancer

## 1. What is a Kubernetes Service?

A Kubernetes Service is an abstraction that exposes an application running on a set of Pods as a network service.

Pods are temporary resources. If a Pod is deleted and recreated, its IP address can change. A Service provides a stable network endpoint and directs traffic to the appropriate backend Pods.

### Example architecture

```text
                 Kubernetes Cluster
        +--------------------------------+
        |                                |
        |       Nginx Deployment         |
        |          /   |   \             |
        |       Pod1  Pod2  Pod3          |
        |          \   |   /             |
        |         Nginx Service           |
        +--------------------------------+
```

A Service commonly selects Pods using labels and selectors.

Example Pod label:

```yaml
metadata:
  labels:
    app: nginx
```

Matching Service selector:

```yaml
spec:
  selector:
    app: nginx
```

The selector allows the Service to identify the Pods that should receive traffic. Kubernetes maintains EndpointSlices for the Service's eligible backends.

## 2. Why do we need different Service types?

Consider an Nginx Deployment with three replicas. The application may need to be accessed in different ways:

- By other applications inside the Kubernetes cluster.
- From outside the cluster for testing.
- From external clients through a cloud load balancer.

The Service type determines the network access pattern. The same Deployment can be exposed through more than one Service, provided the Services select the same Pods.

```text
                 Nginx Deployment
                        |
                 Three Nginx Pods
                        |
             +----------+----------+
             |          |          |
             v          v          v
         ClusterIP    NodePort  LoadBalancer
          Service     Service    Service
```

## 3. ClusterIP Service

### What is ClusterIP?

`ClusterIP` is the default Kubernetes Service type. It exposes a Service on an internal IP address that is normally reachable from within the cluster.

### How it works

1. An application inside the cluster sends a request to the Service's DNS name or ClusterIP.
2. Kubernetes Service networking directs the request to an eligible backend Pod.
3. The Pod processes the request and returns a response.

```text
Frontend Pod
     |
     | http://nginx-service
     v
ClusterIP Service
     |
     +------> Nginx Pod 1
     +------> Nginx Pod 2
     +------> Nginx Pod 3
```

### Access

From a Pod in the same namespace, assuming the Service is named `nginx-service` and listens on port 80:

```text
http://nginx-service
```

A fully qualified Service DNS name can also be used:

```text
http://nginx-service.default.svc.cluster.local
```

From outside the cluster, the ClusterIP is not normally directly reachable.

### Why use ClusterIP?

- Internal communication between microservices.
- Accessing backend APIs or databases from other workloads.
- Providing a stable internal DNS name and virtual IP.

### Example YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

- `port`: port exposed by the Service.
- `targetPort`: port on the selected Pod that receives traffic.
- `selector`: identifies Pods using matching labels.

## 4. NodePort Service

### What is NodePort?

`NodePort` exposes a Service on a port of each node. A client can attempt to reach the application through a node IP address and the assigned NodePort.

The default NodePort range is usually `30000–32767`, unless the cluster is configured with a different range.

### How it works

1. A client sends a request to a reachable node IP and NodePort.
2. Kubernetes networking forwards the traffic to the Service's backend.
3. The request reaches an eligible Pod.

```text
External Client
      |
      | http://<NODE-IP>:30080
      v
Kubernetes Node :30080
      |
      v
NodePort Service
      |
      +------> Nginx Pod 1
      +------> Nginx Pod 2
      +------> Nginx Pod 3
```

### Access

From outside the cluster, if the node is reachable and network/firewall rules permit traffic:

```text
http://<NODE-IP>:30080
```

From inside the cluster, clients can usually use the Service DNS name and Service port:

```text
http://nginx-nodeport
```

### Why use NodePort?

- Learning and testing Kubernetes networking.
- Environments where access through node addresses and ports is suitable.
- As one possible building block for external networking.

### Limitations

- Clients need a reachable node address and the correct port.
- Firewall rules, routing, and node accessibility must permit traffic.
- It does not by itself provide host-based or path-based HTTP routing.

### Example YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

## 5. LoadBalancer Service

### What is LoadBalancer?

`LoadBalancer` exposes a Service through an external load-balancing implementation. In supported cloud environments, a Kubernetes integration or controller typically provisions or configures a cloud load balancer.

The resulting endpoint may be a public or private IP address or DNS name, depending on the configuration.

### How it works

1. You create a Service of type `LoadBalancer`.
2. A supported cloud integration/controller configures a load balancer.
3. A client sends traffic to the load balancer's IP address or DNS name.
4. The configured traffic path forwards requests to the Service's eligible Pods.

```text
External Client
      |
      v
Cloud Load Balancer
      |
      v
LoadBalancer Service
      |
      +------> Nginx Pod 1
      +------> Nginx Pod 2
      +------> Nginx Pod 3
```

### Access

From outside the cluster, after the load balancer is provisioned and network access is permitted:

```text
http://<EXTERNAL-IP-OR-DNS>
```

From inside the cluster, clients can usually use the Service DNS name and Service port:

```text
http://nginx-loadbalancer
```

### Why use LoadBalancer?

- Exposing applications through managed cloud load-balancing infrastructure.
- Providing a stable external endpoint.
- Supporting production access patterns where a cloud load balancer is appropriate.

### Important considerations

- `type: LoadBalancer` does not guarantee a public endpoint. The load balancer may be internal/private.
- Provisioning and behavior depend on the Kubernetes platform and installed controllers.
- Cloud load balancers, networking, and data transfer may incur costs.
- On a local cluster, an external load balancer may not be provisioned unless the environment provides a suitable implementation.

### Example YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

## 6. Compare ClusterIP, NodePort, and LoadBalancer

| Feature | ClusterIP | NodePort | LoadBalancer |
|---|---|---|---|
| Main purpose | Internal cluster access | Access through node IP and port | Access through a load-balancing endpoint |
| Internal access | Yes | Yes | Yes |
| External access | Not normally direct | If a node is reachable and traffic is allowed | If the provisioned endpoint and network rules permit it |
| Typical address | Service DNS or ClusterIP | `http://<NODE-IP>:<NODE-PORT>` | `http://<EXTERNAL-IP-OR-DNS>` |
| Cloud load balancer | No, not by itself | No, not by itself | Usually integrated/provisioned in supported environments |
| Common use | Microservice-to-microservice communication | Testing and specific networking setups | External application access |

## 7. Inside-cluster versus outside-cluster access

Assume the Services are named `nginx-clusterip`, `nginx-nodeport`, and `nginx-loadbalancer`, all expose Service port 80, and the node/load-balancer endpoints are reachable.

| Service | From inside the cluster | From outside the cluster |
|---|---|---|
| ClusterIP | `http://nginx-clusterip` | No normal direct access to the ClusterIP |
| NodePort | `http://nginx-nodeport` | `http://<NODE-IP>:30080` |
| LoadBalancer | `http://nginx-loadbalancer` | `http://<EXTERNAL-IP-OR-DNS>` |

These URLs assume the client is in the same namespace for short DNS names. For another namespace, use an appropriate namespace-qualified DNS name. Replace the example addresses with the values from your own cluster.

External access depends on routing, firewalls, security groups, and the cluster's network configuration.

## 8. Important port concepts

Consider this Service port configuration:

```yaml
ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

- `port: 80`: the port exposed by the Service.
- `targetPort: 8080`: the destination port on the Pod.
- `nodePort: 30080`: the port exposed on each node for a NodePort Service, or where applicable for a LoadBalancer Service using NodePort allocation.

Traffic conceptually follows:

```text
Client
  |
  | Node IP : 30080
  v
Service port : 80
  |
  v
Pod port : 8080
```

`nodePort` is not required in every LoadBalancer configuration; Kubernetes and the load-balancer implementation determine the relevant behavior.

## 9. Do Services replace one another?

No. Each type addresses a different access requirement.

- Choose `ClusterIP` for internal communication.
- Choose `NodePort` when node-IP-and-port access is appropriate.
- Choose `LoadBalancer` when you need an external load-balancing endpoint supported by your platform.

A common web architecture can also use an Ingress Controller for HTTP/HTTPS routing to multiple Services. Ingress does not replace the backend Services.

## 10. Interview answer

**Question: What is the difference between ClusterIP, NodePort, and LoadBalancer?**

Answer:

> ClusterIP is the default Service type and provides internal access to an application through a stable Service IP and DNS name. NodePort exposes the Service on a port of each Kubernetes node, allowing clients to connect through a reachable node IP and port. LoadBalancer integrates with a supported load-balancing implementation to expose the Service through an external or internal load-balancer endpoint. The choice depends on the required access pattern and the cluster environment.

## 11. Key points to remember

- A Deployment manages the Pods; a Service provides a stable network endpoint for reaching them.
- Multiple Services can select the same Pods.
- ClusterIP is intended for internal access.
- NodePort uses a node IP and port for external access when the node is reachable.
- LoadBalancer provides an endpoint through a load-balancing implementation.
- An external endpoint is not necessarily public; networking and security configuration still matter.
- Use the actual IP addresses, ports, and DNS names reported by your cluster when documenting hands-on results.
