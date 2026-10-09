# Kubernetes LoadBalancer Service — Hands-on Lab

## Objective

Expose an application running in Kubernetes through a `LoadBalancer` Service, inspect its status, and test the available access paths.

## Prerequisites

- A Kubernetes cluster is available.
- An Nginx Deployment named `nginx-demo` exists with three replicas.
- The Pods use the label `app: nginx-demo`.
- Nginx listens on port `80`.

Check the resources:

```bash
kubectl get nodes -o wide
kubectl get deployments
kubectl get pods -o wide --show-labels
kubectl get services
```

Ensure that the Nginx Deployment has three ready replicas before proceeding.

## 1. Create the LoadBalancer Service YAML

Create a file named `nginx-loadbalancer-service.yaml`:

```bash
nano nginx-loadbalancer-service.yaml
```

Add this manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx-demo
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

Save the file in Nano with `Ctrl+O`, press `Enter`, then exit with `Ctrl+X`.

### Explanation

- `apiVersion: v1`: API version for a Service.
- `kind: Service`: creates a Kubernetes Service.
- `metadata.name`: names the Service `nginx-loadbalancer`.
- `type: LoadBalancer`: requests integration with a load-balancing implementation.
- `selector.app`: selects Pods labelled `app: nginx-demo`.
- `port: 80`: port exposed by the Service.
- `targetPort: 80`: port on the selected Pods that receives traffic.
- `protocol: TCP`: uses TCP networking.

## 2. Validate and apply the manifest

Validate the manifest:

```bash
kubectl apply --dry-run=client -f nginx-loadbalancer-service.yaml
```

Apply it:

```bash
kubectl apply -f nginx-loadbalancer-service.yaml
```

## 3. Inspect the Service status

```bash
kubectl get svc nginx-loadbalancer
kubectl describe svc nginx-loadbalancer
```

You may see an output similar to:

```text
NAME                 TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)
nginx-loadbalancer   LoadBalancer   10.98.198.8   <pending>     80:31542/TCP
```

Actual addresses and ports vary by cluster.

### Understand the output

- `TYPE: LoadBalancer`: the Service requests load-balancer integration.
- `CLUSTER-IP`: the Service's internal virtual IP.
- `EXTERNAL-IP: <pending>`: an external address has not yet been assigned.
- `PORT(S)`: shows the Service port and, when allocated, its NodePort.

A `LoadBalancer` Service does not guarantee that a public endpoint will be created. A suitable cloud integration or load-balancer controller must be available and configured. The endpoint may be public or private depending on the platform and configuration.

## 4. Check backend endpoints

Run:

```bash
kubectl get endpointslices -l kubernetes.io/service-name=nginx-loadbalancer
```

The EndpointSlice should list the eligible Nginx Pod IP addresses and port `80`.

You can also inspect the Service selector:

```bash
kubectl describe svc nginx-loadbalancer
```

Confirm that the selector is `app=nginx-demo` and that endpoints are present.

## 5. Wait briefly for an external address

Watch the Service:

```bash
kubectl get svc nginx-loadbalancer -w
```

Press `Ctrl+C` to stop watching.

If `EXTERNAL-IP` remains `<pending>`, check whether the cluster has a supported load-balancer controller or cloud integration. Do not assume an external load balancer exists just because the Service object was created successfully.

## 6. Test internal Service access

Even if external provisioning is pending, the Service can still have a working internal ClusterIP and backend endpoints.

If a client Pod named `curl-test` already exists, enter it:

```bash
kubectl exec -it curl-test -- sh
```

Inside the container, run:

```sh
curl -i http://nginx-loadbalancer
```

Expected result: an HTTP success response and the Nginx welcome HTML.

Exit the container:

```sh
exit
```

If you do not have a client Pod, create one:

```bash
kubectl run curl-test \
  --image=curlimages/curl:latest \
  --restart=Never \
  --command -- sleep 1800
```

Wait for it to be ready, then use `kubectl exec` as shown above.

This confirms internal Service connectivity. It does not prove that an external cloud load balancer is functioning.

## 7. Test external access if an address is assigned

If the Service receives an external IP or hostname, inspect it:

```bash
kubectl get svc nginx-loadbalancer
```

When the endpoint is assigned and configured for client access, test it from a machine that can reach that endpoint:

```bash
curl -i http://<EXTERNAL-IP-OR-DNS>
```

You can also open the endpoint in a browser:

```text
http://<EXTERNAL-IP-OR-DNS>
```

Replace the placeholder with the actual address. The load balancer must permit HTTP traffic, and the client must have network connectivity to it. Do not use the ClusterIP as if it were a public address.

## 8. Test the allocated NodePort only if useful

Some LoadBalancer implementations allocate a NodePort as part of the Service. Check the actual port with:

```bash
kubectl get svc nginx-loadbalancer
```

If a NodePort is allocated and a node is reachable, you may test:

```bash
curl -i http://<REACHABLE-NODE-IP>:<NODE-PORT>
```

This tests access through the node and allocated port; it does not prove that the external load balancer itself is working.

## 9. Understand the traffic flow

When external load-balancer provisioning is supported:

```text
External Client
      |
      v
External Load Balancer
      |
      v
LoadBalancer Service
      |
      +------> Nginx Pod 1 :80
      +------> Nginx Pod 2 :80
      +------> Nginx Pod 3 :80
```

The exact traffic path depends on the Kubernetes platform, controller, and configuration.

## 10. Troubleshooting

### External IP remains pending

Run:

```bash
kubectl get svc nginx-loadbalancer
kubectl describe svc nginx-loadbalancer
kubectl get events --sort-by=.metadata.creationTimestamp
```

Check whether the platform has a compatible load-balancer implementation configured. In a local or training cluster, external provisioning may not be available by default.

### No backend endpoints

```bash
kubectl get pods --show-labels
kubectl get endpointslices -l kubernetes.io/service-name=nginx-loadbalancer
kubectl describe svc nginx-loadbalancer
```

Verify that the selector matches the Pod labels and the Pods are ready.

### External address exists but requests fail

Check that:

- The address is reachable from the client.
- The load balancer is configured for the expected protocol and port.
- Firewalls, security groups, routes, and network policies allow the traffic.
- The Service has ready backend endpoints.

## 11. Quick command reference

```bash
kubectl apply --dry-run=client -f nginx-loadbalancer-service.yaml
kubectl apply -f nginx-loadbalancer-service.yaml

kubectl get svc nginx-loadbalancer
kubectl describe svc nginx-loadbalancer
kubectl get endpointslices -l kubernetes.io/service-name=nginx-loadbalancer
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 12. Result checklist

- [ ] LoadBalancer Service YAML was created and applied.
- [ ] Service type and ClusterIP were verified.
- [ ] Backend EndpointSlices were checked.
- [ ] External address status was recorded accurately.
- [ ] Internal connectivity was tested.
- [ ] External access was tested only if a reachable external endpoint was provisioned.
- [ ] Any platform limitation was documented.

## 13. Interview takeaway

> A Kubernetes `LoadBalancer` Service requests exposure through a supported load-balancing implementation. In a cloud environment, a controller or integration may provision an external or internal load balancer. The Service can still have a ClusterIP and backend endpoints while the external address remains pending. Actual external access depends on platform support, load-balancer provisioning, and network/security configuration.
