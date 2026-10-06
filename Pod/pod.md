# Kubernetes Pod Lifecycle

## Objective

Understand the Kubernetes Pod lifecycle by creating a short-lived Pod and observing its lifecycle using:

```bash
kubectl get pod -w
```

The main Pod lifecycle phases are:

```text
Pending → Running → Succeeded
```

A Pod can also reach:

```text
Failed
```

when its containers terminate unsuccessfully.

---

## 1. Create a Short-Lived Pod

Create a file:

```bash
vi pod-lifecycle.yaml
```

Add the following manifest:

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

### Explanation

- `apiVersion: v1` → Uses the Kubernetes core API.
- `kind: Pod` → Creates a Pod.
- `metadata.name` → Pod name is `lifecycle-pod`.
- `image: busybox` → Uses the lightweight BusyBox container image.
- `command` → Executes a shell command inside the container.
- `echo Hello Kubernetes` → Prints a message.
- `sleep 5` → Keeps the container running for 5 seconds and then exits successfully.

---

## 2. Create the Pod

Apply the manifest:

```bash
kubectl apply -f pod-lifecycle.yaml
```

Expected:

```text
pod/lifecycle-pod created
```

---

## 3. Watch the Pod Lifecycle

Run:

```bash
kubectl get pod lifecycle-pod -w
```

The Pod may move through states similar to:

```text
Pending
   ↓
Running
   ↓
Completed
```

The actual Kubernetes Pod phase for `Completed` is:

```text
Succeeded
```

Use `Ctrl+C` to stop watching.

---

## 4. Verify the Final Pod Phase

Run:

```bash
kubectl get pod lifecycle-pod -o jsonpath='{.status.phase}'
```

Expected:

```text
Succeeded
```

You can also run:

```bash
kubectl get pods
```

Example:

```text
NAME             READY   STATUS      RESTARTS   AGE
lifecycle-pod    0/1     Completed   0          20s
```

> `kubectl get pods` commonly displays `Completed`, but the actual Kubernetes Pod phase is `Succeeded`.

---

# Pod Lifecycle Phases

## Pending

The Pod has been accepted by Kubernetes but is not yet running all of its containers.

Possible reasons include:

- Pod waiting for scheduling
- Container image being pulled
- Volume or networking setup

```text
Pending
```

---

## Running

The Pod has been scheduled to a node and its containers are running or starting.

```text
Running
```

---

## Succeeded

All containers in the Pod have terminated successfully.

For example:

```bash
echo "Hello Kubernetes"
sleep 5
```

After the command completes successfully, the container exits with exit code `0`.

Therefore:

```text
Running
   ↓
Task completed successfully
   ↓
Succeeded
```

---

## Failed

The containers in the Pod terminated unsuccessfully.

For example, a command can intentionally fail:

```yaml
command: ["sh", "-c", "echo Test; exit 1"]
```

The container exits with a non-zero exit code, and the Pod can reach:

```text
Failed
```

---

# Lifecycle Flow

```text
             Pod Created
                  |
                  v
              Pending
                  |
                  v
              Running
               /     \
              /       \
             v         v
       Succeeded     Failed
       (success)     (failure)
```

---

# Important Kubernetes Commands

### Create/Apply Pod

```bash
kubectl apply -f pod-lifecycle.yaml
```

### Watch lifecycle

```bash
kubectl get pod lifecycle-pod -w
```

### Check Pod status

```bash
kubectl get pods
```

### Check actual Pod phase

```bash
kubectl get pod lifecycle-pod -o jsonpath='{.status.phase}'
```

### Detailed information

```bash
kubectl describe pod lifecycle-pod
```

### Delete the Pod

```bash
kubectl delete pod lifecycle-pod
```

---

# Key Learning

- **Pending** → Pod accepted but not running yet.
- **Running** → Container(s) are running.
- **Succeeded** → Containers completed successfully.
- **Failed** → Containers terminated unsuccessfully.
- `kubectl get pods` may display **Completed**, but the actual Pod phase is **Succeeded**.
- Short-lived Pods are useful for understanding batch tasks and are commonly managed by higher-level resources such as **Jobs**.
- `kubectl get pod -w` is useful for observing state changes in real time.
