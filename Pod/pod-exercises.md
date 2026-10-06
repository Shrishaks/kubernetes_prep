# Kubernetes Pod — Hands-on Exercises

This document contains four hands-on exercises covering basic Kubernetes Pod creation, deletion, shared storage, and Pod lifecycle.

---

# Exercise 1 — Create an NGINX Pod Using a Manifest

## Objective

Create an NGINX Pod using a YAML manifest instead of `kubectl run`, and verify the Pod using:

```bash
kubectl get pods -o wide
kubectl describe pod
```

## 1. Create the Pod Manifest

Create `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

## 2. Apply the Manifest

```bash
kubectl apply -f pod.yaml
```

Expected:

```text
pod/nginx-pod created
```

## 3. Verify the Pod

```bash
kubectl get pods -o wide
```

The `-o wide` option provides additional information such as:

- Pod IP address
- Node where the Pod is running
- Status
- Restart count

Example:

```text
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE
nginx-pod    1/1     Running   0          30s   10.244.0.5   worker-node
```

## 4. Describe the Pod

```bash
kubectl describe pod nginx-pod
```

This provides detailed information about:

- Pod metadata
- Node
- IP address
- Containers
- Images
- Container state
- Conditions
- Volumes
- Events

## Key Learning

A Pod can be created declaratively using a YAML manifest:

```text
YAML Manifest
     ↓
kubectl apply
     ↓
Pod
     ↓
kubectl get / describe
```

---

# Exercise 2 — Delete a Bare Pod and Confirm It Is Not Recreated

## Objective

Create/delete a bare Pod and understand why Kubernetes does not recreate it after deletion.

## 1. Delete the Pod

For example:

```bash
kubectl delete pod nginx-pod
```

Expected:

```text
pod "nginx-pod" deleted
```

## 2. Verify

```bash
kubectl get pods
```

The Pod should no longer exist.

It will **not be recreated**.

## Why?

The Pod was created directly:

```text
User
 ↓
Pod
```

There is no higher-level controller managing it.

### kubelet

The kubelet runs on each Kubernetes node and makes sure the containers for assigned Pods are running.

However:

> kubelet does not recreate a deleted Pod object.

### Deployment / ReplicaSet

A Deployment creates and manages a ReplicaSet.

The ReplicaSet maintains the desired number of Pods.

For example:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

If the Pod is deleted:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod ❌
    ↓
ReplicaSet detects missing Pod
    ↓
New Pod ✅
```

## Key Learning

A **bare Pod is not self-healing**.

Higher-level controllers such as Deployments and ReplicaSets provide the desired-state management that recreates Pods when necessary.

---

# Exercise 3 — Two Containers Sharing an `emptyDir` Volume

## Objective

Create a Pod with two containers sharing the same `emptyDir` volume.

- The `writer` container writes a file.
- The `reader` container reads the same file.

## 1. Create the Manifest

Create `shared-volume-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  volumes:
    - name: shared-data
      emptyDir: {}

  containers:
    - name: writer
      image: busybox
      command: ["/bin/sh", "-c"]
      args:
        - echo "Hello from writer container" > /shared/data.txt;
          sleep 3600
      volumeMounts:
        - name: shared-data
          mountPath: /shared

    - name: reader
      image: busybox
      command: ["/bin/sh", "-c"]
      args:
        - sleep 10;
          cat /shared/data.txt;
          sleep 3600
      volumeMounts:
        - name: shared-data
          mountPath: /shared
```

## 2. Apply the Manifest

```bash
kubectl apply -f shared-volume-pod.yaml
```

Check the Pod:

```bash
kubectl get pod shared-volume-pod
```

Expected:

```text
NAME                 READY   STATUS
shared-volume-pod    2/2     Running
```

## 3. Verify the Writer

```bash
kubectl exec shared-volume-pod -c writer -- cat /shared/data.txt
```

Expected:

```text
Hello from writer container
```

## 4. Verify the Reader

```bash
kubectl exec shared-volume-pod -c reader -- cat /shared/data.txt
```

Expected:

```text
Hello from writer container
```

This proves both containers are accessing the same volume.

## How It Works

The Pod defines one volume:

```yaml
volumes:
  - name: shared-data
    emptyDir: {}
```

Both containers mount the same volume:

```yaml
volumeMounts:
  - name: shared-data
    mountPath: /shared
```

Therefore:

```text
                  Pod
                   |
             emptyDir volume
              shared-data
                   |
          ┌────────┴────────┐
          │                 │
       Writer             Reader
       /shared            /shared
          │                 │
          ↓                 ↓
       writes            reads
     data.txt            data.txt
```

## Important Concept

### `volumes`

Defines the storage:

```yaml
volumes:
  - name: shared-data
    emptyDir: {}
```

### `volumeMounts`

Mounts the storage inside a container:

```yaml
volumeMounts:
  - name: shared-data
    mountPath: /shared
```

### `emptyDir`

`emptyDir` is temporary Pod-level storage.

The data remains when a container restarts, as long as the Pod still exists.

If the entire Pod is deleted, the `emptyDir` data is deleted.

---

# Exercise 4 — Pod Lifecycle

## Objective

Observe the lifecycle of a short-lived Pod using:

```bash
kubectl get pod -w
```

The main Pod phases are:

```text
Pending → Running → Succeeded
```

A Pod can also reach:

```text
Failed
```

when its containers terminate unsuccessfully.

## 1. Create the Manifest

Create `pod-lifecycle.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: ["sh", "-c", "echo Hello Kubernetes; sleep 5"]
```

## 2. Apply the Manifest

```bash
kubectl apply -f pod-lifecycle.yaml
```

## 3. Watch the Lifecycle

Run:

```bash
kubectl get pod lifecycle-pod -w
```

The Pod may transition through:

```text
Pending
   ↓
Running
   ↓
Completed
```

`kubectl get pods` commonly displays:

```text
Completed
```

But the actual Kubernetes Pod phase is:

```text
Succeeded
```

## 4. Check the Actual Pod Phase

```bash
kubectl get pod lifecycle-pod -o jsonpath='{.status.phase}'
```

Expected:

```text
Succeeded
```

## Pod Lifecycle Phases

### Pending

The Pod has been accepted by Kubernetes but is not yet running all of its containers.

Possible reasons:

- Scheduling
- Image pulling
- Volume setup
- Network setup

```text
Pending
```

### Running

The Pod has been scheduled to a node and its containers are running or starting.

```text
Running
```

### Succeeded

All containers completed successfully.

For example:

```bash
echo "Hello Kubernetes"
sleep 5
```

The command finishes successfully and the container exits with code `0`.

```text
Running
   ↓
Successful completion
   ↓
Succeeded
```

### Failed

The container terminates unsuccessfully.

For example:

```yaml
command: ["sh", "-c", "echo Test; exit 1"]
```

The container exits with a non-zero exit code.

The Pod can then reach:

```text
Failed
```

---

# Pod Lifecycle Diagram

```text
                 Pod Created
                      |
                      ↓
                   Pending
                      |
                      ↓
                   Running
                   /      \
                  /        \
                 ↓          ↓
            Succeeded     Failed
             Success      Failure
```

---

# Important Commands Used

## Create/Apply

```bash
kubectl apply -f pod.yaml
```

## Get Pods

```bash
kubectl get pods
```

## Get Pods with Additional Information

```bash
kubectl get pods -o wide
```

## Watch Pod Changes

```bash
kubectl get pod <pod-name> -w
```

## Describe Pod

```bash
kubectl describe pod <pod-name>
```

## Execute a Command in a Container

```bash
kubectl exec <pod-name> -c <container-name> -- <command>
```

## Delete a Pod

```bash
kubectl delete pod <pod-name>
```

## Check Actual Pod Phase

```bash
kubectl get pod <pod-name> -o jsonpath='{.status.phase}'
```

---

# Key Takeaways

1. A Pod can be created using a declarative YAML manifest.
2. `kubectl apply -f` is commonly used to apply the manifest.
3. `kubectl get pods -o wide` provides additional Pod information including IP and Node.
4. `kubectl describe pod` provides detailed Pod information and events.
5. A bare Pod is not automatically recreated after deletion.
6. Controllers such as ReplicaSets and Deployments provide Pod reconciliation and self-healing.
7. Multiple containers in the same Pod can share an `emptyDir` volume.
8. `volumes` defines storage, while `volumeMounts` attaches it to containers.
9. `emptyDir` data exists for the lifetime of the Pod.
10. Pod lifecycle phases include `Pending`, `Running`, `Succeeded`, and `Failed`.
11. `kubectl get pod -w` can be used to observe Pod state changes in real time.
12. `Completed` shown by `kubectl get pods` corresponds to the Pod phase `Succeeded`.
