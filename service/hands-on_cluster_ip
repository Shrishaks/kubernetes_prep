# Kubernetes ClusterIP Service — Hands-on Lab

## Objective

Deploy an Nginx application with three replicas in Iximiuz Labs, expose it using a Kubernetes `ClusterIP` Service, and verify that the application is reachable from another Pod inside the cluster.

## 1. Create the Nginx Deployment

Create `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
```

Save the file in Nano with `Ctrl+O`, press `Enter`, then exit with `Ctrl+X`.

Validate and apply:

```bash
kubectl apply --dry-run=client -f nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get pods -o wide
```

Expected state: Deployment `nginx-demo` has three ready replicas, and all three Nginx Pods are `Running` and `Ready`.

### Important YAML fields

- `apiVersion: apps/v1`: Deployment API version.
- `kind: Deployment`: asks Kubernetes to manage replicated Pods.
- `replicas: 3`: desired number of Pods.
- `selector.matchLabels`: identifies Pods managed by the Deployment.
- `template.metadata.labels`: assigns the matching label to each Pod.
- `image: nginx:stable`: specifies the Nginx image.
- `containerPort: 80`: documents the container port; it does not expose the application externally.

## 2. Create the ClusterIP Service

Create `nginx-clusterip-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx-demo
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

Apply and inspect:

```bash
kubectl apply -f nginx-clusterip-service.yaml
kubectl get svc nginx-clusterip
kubectl get endpointslices -l kubernetes.io/service-name=nginx-clusterip
```

Observed in this lab:

- Service type: `ClusterIP`
- Service port: `80/TCP`
- ClusterIP at the time of testing: `10.104.253.93`
- Endpoint IPs: `10.244.2.2`, `10.244.0.2`, and `10.244.1.4`

These are session-specific values and can differ in another cluster.

### How the Service selects Pods

The Service selector is `app: nginx-demo`. The Deployment's Pod template assigns the same label, so the Service selects the three Nginx Pods. Kubernetes publishes eligible backend addresses through EndpointSlices.

## 3. Create a temporary client Pod

To test internal access interactively, create a client Pod with `curl` installed:

```bash
kubectl run curl-test \
  --image=curlimages/curl:latest \
  --restart=Never \
  --command -- sleep 1800
```

Wait until it is ready:

```bash
kubectl get pods
```

`sleep 1800` keeps the container running for up to 30 minutes so you can run commands interactively. This is a standalone troubleshooting Pod, not part of the Nginx Deployment.

Enter the client container:

```bash
kubectl exec -it curl-test -- sh
```

## 4. Test ClusterIP access from inside the cluster

Inside the `curl-test` container, run:

```sh
curl http://nginx-clusterip
```

Expected result: HTML output containing the Nginx welcome page, including:

```html
<title>Welcome to nginx!</title>
<h1>Welcome to nginx!</h1>
```

Exit the container shell:

```sh
exit
```

### What the successful test proves

```text
curl-test Pod
     |
     | curl http://nginx-clusterip
     v
ClusterIP Service
     |
     +------> Nginx Pod 1
     +------> Nginx Pod 2
     +------> Nginx Pod 3
```

The request travelled from the client Pod to the Service DNS name, then to one of the selected Nginx Pods. This confirms internal Service connectivity; it does not test access from outside the cluster.

## 5. Why ClusterIP is useful

Pods can be replaced, and their IP addresses can change. A Service provides a stable internal IP and DNS name so clients do not have to depend on individual Pod IP addresses.

Typical uses include:

- Frontend-to-backend communication.
- Internal APIs and microservices.
- Internal database access.
- Service discovery within a cluster.

A ClusterIP Service is intended for internal cluster access and is not normally directly reachable from a user's machine outside the cluster.

## 6. Troubleshooting commands

```bash
kubectl get deployments
kubectl describe deployment nginx-demo
kubectl get pods -o wide --show-labels
kubectl describe svc nginx-clusterip
kubectl get endpointslices -l kubernetes.io/service-name=nginx-clusterip
```

If no endpoints appear, check that the Service selector matches the Pod labels and that the Pods are ready. If curl fails, verify the client Pod is running, the Service name is correct, and the Service port and `targetPort` match the application.

## 7. Commands used — quick reference

```bash
kubectl get nodes -o wide
kubectl get deployments
kubectl get services

kubectl apply --dry-run=client -f nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml
kubectl get deployments
kubectl get pods -o wide

kubectl apply -f nginx-clusterip-service.yaml
kubectl get svc nginx-clusterip
kubectl get endpointslices -l kubernetes.io/service-name=nginx-clusterip

kubectl run curl-test --image=curlimages/curl:latest --restart=Never --command -- sleep 1800
kubectl get pods
kubectl exec -it curl-test -- sh
# Inside the client container:
curl http://nginx-clusterip
exit
```

## 8. Result

**Status: Successful.**

The Nginx Deployment ran three ready replicas. The ClusterIP Service selected all three backend Pods, and a request from the `curl-test` Pod to `http://nginx-clusterip` returned the Nginx welcome HTML.

## 9. Interview takeaway

> A ClusterIP Service is the default Kubernetes Service type. It provides a stable internal IP and DNS name for accessing selected Pods. In this lab, I deployed Nginx with three replicas, created a ClusterIP Service using a label selector, verified the backend EndpointSlice addresses, and tested connectivity from a separate client Pod using `curl`.

## Next lab

Continue with the same Deployment and create a `NodePort` Service. Then create a `LoadBalancer` Service and document whether the lab environment provisions an external load-balancer address.
