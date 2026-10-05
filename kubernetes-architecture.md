# Kubernetes Cluster Architecture

Kubernetes follows a **Control Plane + Worker Node** architecture.

The Control Plane manages the Kubernetes cluster, while Worker Nodes run the application Pods.

## Architecture

```text
                         Kubernetes Cluster
                                │
              ┌─────────────────┴─────────────────┐
              │                                   │
              ▼                                   ▼
       ┌───────────────┐                 ┌──────────────────┐
       │ Control Plane │                 │   Worker Nodes   │
       └───────────────┘                 └──────────────────┘
              │                                   │
       ┌──────┼─────────────┐             ┌───────┼────────┐
       │      │             │             │       │        │
       ▼      ▼             ▼             ▼       ▼        ▼
   API Server etcd    Controller       kubelet kube-proxy  CNI
   :6443             Manager            │                  │
       │                               ▼                  │
       │                           containerd         Calico/
       │                               │              Flannel
       │                               ▼
       │                              Pods
       │
       ▼
   Scheduler
```

---

# 1. Control Plane

The Control Plane is responsible for **managing the Kubernetes cluster**.

It contains the following major components:

### kube-apiserver

The **API Server** is the entry point to the Kubernetes cluster.

- Kubernetes API communication happens through the API Server.
- `kubectl` communicates with the API Server.
- Default secure API port is **6443**.
- It authenticates, authorizes, validates, and processes API requests.
- Other Kubernetes components communicate through the API Server.

```text
kubectl
   │
   │ HTTPS :6443
   ▼
kube-apiserver
```

---

### etcd

**etcd** is the distributed key-value store used by Kubernetes.

It stores important cluster information such as:

- Cluster state
- Configuration
- Kubernetes objects
- Metadata
- Desired state

Example:

```text
Deployment
    ↓
3 replicas
    ↓
nginx image
```

The Kubernetes cluster state is stored in etcd.

> **Important:** etcd backup is an important part of Kubernetes disaster recovery because it contains the cluster state.

---

### Controller Manager

The **Controller Manager** continuously watches the cluster and works to make the **actual state match the desired state**.

For example:

```text
Desired state = 3 Pods

Current state = 2 Pods
```

The controller detects the difference and works toward:

```text
Current state → 3 Pods
```

Think of it as:

> **Controller Manager = Maintains the desired state**

---

### Scheduler

The **Scheduler** decides which Worker Node should run a newly created Pod.

It considers factors such as:

- CPU and memory availability
- Resource requests
- Node selectors
- Affinity / anti-affinity
- Taints and tolerations
- Other scheduling constraints

Example:

```text
New Pod
   │
   ▼
Scheduler
   │
   ▼
Worker Node 2
```

Think of it as:

> **Scheduler = Decides WHERE the Pod should run**

---

# 2. Worker Node

Worker Nodes are responsible for **running application workloads**.

Each Worker Node normally contains:

- kubelet
- Container Runtime
- kube-proxy
- CNI networking
- Pods

---

## kubelet

The **kubelet** is the Kubernetes agent running on every Worker Node.

It:

- Communicates with the API Server
- Receives Pod specifications
- Ensures assigned Pods are running
- Monitors Pod health
- Works with the container runtime

Basic flow:

```text
API Server
     │
     ▼
  kubelet
     │
     ▼
Container Runtime
     │
     ▼
    Pod
```

Think of it as:

> **kubelet = Agent responsible for Pods on the Worker Node**

---

# 3. Container Runtime

The container runtime is responsible for running containers.

A common Kubernetes container runtime is:

```text
containerd
```

The kubelet communicates with the container runtime through the **Container Runtime Interface (CRI)**.

The runtime:

- Pulls container images
- Creates containers
- Starts containers
- Stops containers
- Manages the container lifecycle

Flow:

```text
kubelet
   │
   │ CRI
   ▼
containerd
   │
   ▼
Container
```

---

# 4. kube-proxy

**kube-proxy** runs on Worker Nodes and helps implement Kubernetes **Service networking and traffic routing**.

Example:

```text
Client
   │
   ▼
Kubernetes Service
   │
   ▼
kube-proxy
   │
   ▼
Pod
```

Depending on the configuration, kube-proxy can use mechanisms such as:

- iptables
- IPVS

> **
<img width="1536" height="1024" alt="kubernetes_Architecture" src="https://github.com/user-attachments/assets/b3527a83-19e0-4986-b043-ed3d45043c30" />
