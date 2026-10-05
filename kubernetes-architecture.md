# Kubernetes Architecture

Kubernetes follows a **Control Plane + Worker Node** architecture.

The **Control Plane** manages the Kubernetes cluster, while **Worker Nodes** run the application Pods.

---

## Architecture

```text
                         Kubernetes Cluster
                                │
              ┌─────────────────┴─────────────────┐
              │                                   │
              ▼                                   ▼
       ┌───────────────┐                 ┌──────────────────┐
       │ Control Plane │                 │   Worker Node    │
       └───────────────┘                 └──────────────────┘
              │                                   │
       ┌──────┼─────────────┐              ┌──────┼──────────┐
       │      │             │              │      │          │
       ▼      ▼             ▼              ▼      ▼          ▼
   API Server  etcd   Controller        kubelet kube-proxy Container
     :6443             Manager                    Runtime
       │                    │                         │
       │                    ▼                         ▼
       │                Scheduler                    Pods
       │
       └──────────── API Communication ────────────────┘
```

---

# 1. Control Plane

The **Control Plane** is responsible for managing the Kubernetes cluster.

It contains the following major components:

- kube-apiserver
- etcd
- Controller Manager
- Scheduler

---

## kube-apiserver

The **kube-apiserver** is the entry point to the Kubernetes cluster.

It:

- Provides the Kubernetes API
- Receives requests from `kubectl`
- Authenticates and authorizes requests
- Validates API requests
- Communicates with other Kubernetes components

The default secure Kubernetes API Server port is:

```text
6443
```

Basic flow:

```text
kubectl
   │
   │ HTTPS :6443
   ▼
kube-apiserver
```

### Remember

> **API Server = Entry point and communication hub**

---

# 2. etcd

**etcd** is the distributed key-value store used by Kubernetes.

It stores important Kubernetes cluster information such as:

- Cluster state
- Configuration
- Kubernetes objects
- Metadata
- Desired state

Example:

```text
Deployment
    │
    ├── Replicas: 3
    └── Image: nginx
```

The Kubernetes cluster state is stored in etcd.

### Disaster Recovery

etcd is very important for Kubernetes disaster recovery because it contains the cluster state.

Regular **etcd backups** are therefore important.

### Remember

> **etcd = Cluster State / Database**

---

# 3. Controller Manager

The **Controller Manager** continuously watches the cluster and works to make the **actual state match the desired state**.

For example:

```text
Desired State = 3 Pods

Actual State = 2 Pods
```

The controller detects the difference and works toward:

```text
Actual State → Desired State
```

### Remember

> **Controller Manager = Maintains the desired state**

---

# 4. Scheduler

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
Worker Node
```

### Remember

> **Scheduler = Decides WHERE the Pod should run**

---

# 5. Worker Node

Worker Nodes are responsible for running application workloads.

A Worker Node commonly contains:

- kubelet
- Container Runtime
- kube-proxy
- Pods

---

## kubelet

The **kubelet** is the Kubernetes agent running on every Worker Node.

It:

- Communicates with the API Server
- Receives Pod specifications
- Ensures assigned Pods are running
- Monitors Pod health
- Works with the Container Runtime

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

### Remember

> **kubelet = Agent responsible for Pods on the Worker Node**

---

# 6. Container Runtime

The **Container Runtime** is responsible for running containers.

A common Kubernetes container runtime is:

```text
containerd
```

The kubelet communicates with the Container Runtime through the **Container Runtime Interface (CRI)**.

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

### Remember

> **Container Runtime = Runs the Containers**

---

# 7. kube-proxy

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

### Remember

> **kube-proxy = Service Traffic Routing**

---

# 8. How Kubernetes Creates a Pod

Suppose we execute:

```bash
kubectl create deployment nginx --image=nginx
```

The request flows through the Kubernetes architecture.

### Step 1 — kubectl

`kubectl` sends the request to the API Server.

```text
kubectl
   │
   ▼
API Server :6443
```

---

### Step 2 — API Server

The API Server processes the request and stores the desired state in etcd.

```text
kubectl
   │
   ▼
API Server
   │
   ▼
etcd
```

---

### Step 3 — Controller Manager

The Controller Manager detects that the Deployment requires Pods.

```text
Deployment
    │
    ▼
Controller Manager
    │
    ▼
Pod requirement
```

---

### Step 4 — Scheduler

The Scheduler selects a suitable Worker Node.

```text
Pod
 │
 ▼
Scheduler
 │
 ▼
Worker Node
```

---

### Step 5 — kubelet

The kubelet on the selected Worker Node receives the Pod specification.

```text
API Server
    │
    ▼
 kubelet
```

---

### Step 6 — Container Runtime

The kubelet works with the Container Runtime to run the container.

```text
kubelet
   │
   ▼
containerd
   │
   ▼
nginx container
```

---

### Step 7 — Pod Runs

The container is now running inside the Pod.

```text
Worker Node
     │
     ▼
   kubelet
     │
     ▼
 containerd
     │
     ▼
    Pod
     │
     ▼
 nginx container
```

---

# Complete Request Flow

```text
                         USER
                           │
                           ▼
                        kubectl
                           │
                           │ HTTPS :6443
                           ▼
                   ┌─────────────────┐
                   │  kube-apiserver │
                   └────────┬────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
            etcd       Controller      Scheduler
                         Manager           │
                                          │
                                          ▼
                                ┌──────────────────┐
                                │   Worker Node    │
                                │                  │
                                │     kubelet      │
                                │        │         │
                                │        ▼         │
                                │    containerd    │
                                │        │         │
                                │        ▼         │
                                │       Pod        │
                                │                  │
                                │   kube-proxy     │
                                │                  │
                                │ Service Routing  │
                                └──────────────────┘
```

---

# The Easiest Way to Remember

Remember the Kubernetes architecture using these simple questions:

| Component | Ask Yourself | Answer |
|---|---|---|
| **kubectl** | How do I talk to Kubernetes? | Client |
| **API Server** | Where does the request go? | API Server |
| **etcd** | Where is cluster state stored? | etcd |
| **Controller Manager** | Who maintains the desired state? | Controller Manager |
| **Scheduler** | Who decides where the Pod runs? | Scheduler |
| **kubelet** | Who manages the Pod on the node? | kubelet |
| **Container Runtime** | Who runs the container? | containerd |
| **kube-proxy** | Who handles Service traffic? | kube-proxy |

### Easy Memory Formula

```text
API Server → Communication
etcd       → State
Controller → Desired State
Scheduler  → Selects Node
kubelet    → Manages Pod
containerd → Runs Container
kube-proxy → Service Traffic
```

Or remember it as:

```text
       TALK
        ↓
   API SERVER
        ↓
      STORE
        ↓
      etcd
        ↓
     CONTROL
        ↓
   Controller
        ↓
     SCHEDULE
        ↓
   Scheduler
        ↓
      NODE
        ↓
    kubelet
        ↓
      RUN
        ↓
   containerd
        ↓
       POD
```

---

# Quick Interview Summary

> **Kubernetes uses a Control Plane to manage the cluster and Worker Nodes to run workloads. `kubectl` communicates with the API Server on port 6443. etcd stores the cluster state, Controller Manager maintains the desired state, and the Scheduler selects a suitable Worker Node for new Pods. The kubelet on the Worker Node manages the Pod and works with the Container Runtime such as containerd to run the containers. kube-proxy handles Kubernetes Service traffic routing.**

---
<img width="1536" height="1024" alt="kubernetes_architecture" src="https://github.com/user-attachments/assets/7a25887b-dab0-4736-908a-c82b596bedb8" />
