# Kubernetes Architecture

Kubernetes is a container orchestration platform used to deploy, manage,
scale, and maintain containerized applications.

At a high level, a Kubernetes cluster consists of two main parts:

1. Control Plane — manages the cluster
2. Worker Nodes — run the application workloads

## Control Plane

### API Server
The central communication point of Kubernetes.

### etcd
A distributed key-value store that stores Kubernetes cluster state.

### Scheduler
Selects a suitable Worker Node for a new Pod.

### Controller Manager
Runs controllers that maintain the desired state of the cluster.

## Worker Node

### Kubelet
An agent that manages Pods assigned to the Worker Node.

### Container Runtime
Runs the containers inside Pods.

Examples:
- containerd
- CRI-O

### kube-proxy
Helps implement Kubernetes Service networking.

### Pod
The smallest deployable unit in Kubernetes.

## Simple Architecture

```text
                    Kubernetes Cluster
                           |
             +-------------+-------------+
             |                           |
       CONTROL PLANE                WORKER NODE
       Manages cluster              Runs workloads
             |                           |
      +------+------+              +-----+------+
      |      |      |              |     |      |
    API    etcd  Scheduler       Kubelet Runtime kube-proxy
   Server
      |
 Controller
 Manager
                                      |
                                      v
                                     Pod
                                      |
                                  Container
🔑 Easy way to remember
Control Plane → Decides and manages
Worker Node → Runs the workload
API Server       → Communication
etcd             → Cluster state
Scheduler        → Selects the node
Controller       → Maintains desired state
Kubelet          → Manages Pods
Runtime          → Runs containers
kube-proxy       → Service networking
Pod              → Runs application containers

Understanding this architecture makes it much easier to understand the Kubernetes components that come next.
#Kubernetes #DevOps #AWS #EKS #CloudNative #KubernetesArchitecture

<img width="1536" height="1024" alt="Kubernetes_architecture" src="https://github.com/user-attachments/assets/817183b5-0ea4-4040-832c-ed7ab3d22300" />
