# Kubernetes Architecture

Kubernetes is a container orchestration platform used to deploy, manage, scale, and maintain containerized applications.

At a high level, a Kubernetes cluster consists of two main parts:

1. **Control Plane** — manages the cluster
2. **Worker Nodes** — run the application workloads

## Control Plane

The Control Plane is responsible for managing the overall state of the Kubernetes cluster.

### API Server

The central communication point of Kubernetes. Tools such as `kubectl` communicate with the cluster through the API Server.

### etcd

A distributed key-value store that stores the Kubernetes cluster state and configuration.

### Scheduler

Selects a suitable Worker Node for a newly created Pod based on available resources and scheduling requirements.

### Controller Manager

Runs controllers that continuously compare the desired state with the actual state and take corrective action when required.

---

## Worker Node

Worker Nodes are where the application workloads actually run.

### Kubelet

An agent running on each Worker Node. It ensures that the Pods assigned to the node are running as expected.

### Container Runtime

Responsible for running the containers inside Pods.

Examples:

- `containerd`
- `CRI-O`

### kube-proxy

Helps implement Kubernetes Service networking and route Service traffic toward the appropriate Pods.

### Pod

The smallest deployable unit in Kubernetes. A Pod contains one or more containers that run the application workload.

---

## Simple Architecture

```text
                         KUBERNETES CLUSTER
                                |
              +-----------------+-----------------+
              |                                   |
        CONTROL PLANE                        WORKER NODE
        Manages cluster                     Runs workloads
              |                                   |
       +------+------+                    +-------+-------+
       |      |      |                    |       |       |
     API    etcd  Scheduler            Kubelet Runtime kube-proxy
    Server
       |
 Controller
 Manager
                                                |
                                                v
                                               Pod
                                                |
                                            Container
```

### Kubernetes Architecture Diagram

![Kubernetes Architecture](https://github.com/user-attachments/assets/817183b5-0ea4-4040-832c-ed7ab3d22300)

---

## 🔑 Easy Way to Remember

### Control Plane vs Worker Node

- **Control Plane →** Decides and manages
- **Worker Node →** Runs the workload

### Kubernetes Components

| Component | Easy way to remember |
|---|---|
| **API Server** | Communication |
| **etcd** | Cluster state |
| **Scheduler** | Selects the node |
| **Controller Manager** | Maintains desired state |
| **Kubelet** | Manages Pods |
| **Container Runtime** | Runs containers |
| **kube-proxy** | Service networking |
| **Pod** | Smallest deployable unit |

### Simple Memory Flow

```text
API Server
    ↓
Communication

etcd
    ↓
Cluster State

Scheduler
    ↓
Selects Node

Controller Manager
    ↓
Maintains Desired State

Kubelet
    ↓
Manages Pods

Container Runtime
    ↓
Runs Containers

kube-proxy
    ↓
Service Networking

Pod
    ↓
Runs Application Containers
```

---

## Key Concept

The simplest way to remember Kubernetes architecture is:

```text
CONTROL PLANE
     ↓
Manages and makes decisions
     ↓
WORKER NODE
     ↓
Runs application workloads
     ↓
POD
     ↓
CONTAINER
```

Understanding this architecture makes it easier to understand the Kubernetes components and workloads that come next.
