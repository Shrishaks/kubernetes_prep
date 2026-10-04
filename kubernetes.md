Kubernetes Architecture 
Kubernetes is a container orchestration platform used to deploy, manage, scale, and maintain containerized applications.
At a high level, a Kubernetes cluster consists of two main parts:
1. Control Plane — manages the cluster
2. Worker Nodes — run the applications
🔹 Control Plane
The Control Plane is responsible for managing the overall state of the Kubernetes cluster.
API Server
The central communication point of Kubernetes. Tools such as kubectl communicate with the cluster through the API Server.
etcd
A distributed key-value store that keeps the Kubernetes cluster state and configuration.
Scheduler
Decides which suitable Worker Node should run a newly created Pod based on available resources and scheduling requirements.
Controller Manager
Runs controllers that continuously compare the desired state with the actual state and take corrective action when required.
🔹 Worker Node
Worker Nodes are where application workloads actually run.
Kubelet
An agent running on each Worker Node. It ensures that the Pods assigned to the node are running as expected.
Container Runtime
Responsible for actually running the containers inside Pods. Examples include containerd and CRI-O.
kube-proxy
Helps implement Kubernetes Service networking and route Service traffic toward the appropriate Pods.
Pod
The smallest deployable unit in Kubernetes. A Pod contains one or more containers that run the application workload.
🔹 Simple Architecture
                    Kubernetes Cluster
                           │
             ┌─────────────┴─────────────┐
             │                           │
       CONTROL PLANE                 WORKER NODE
        Manages cluster              Runs workloads
             │                           │
      ┌──────┼──────┐             ┌──────┼──────┐
      │      │      │             │      │      │
    API     etcd  Scheduler    Kubelet Runtime kube-proxy
   Server
      │
 Controller
 Manager
                                      │
                                      ▼
                                     Pod
                                      │
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
