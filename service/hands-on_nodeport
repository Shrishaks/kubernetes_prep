# Kubernetes NodePort Service — Hands-on Lab

## Objective

Expose an application running in Kubernetes using a `NodePort` Service and verify access through a node IP address and a designated port.

## Prerequisites

- A Kubernetes cluster is available.
- An Nginx Deployment named `nginx-demo` exists with three replicas.
- The Deployment's Pods have the label `app: nginx-demo`.
- The Nginx container listens on port `80`.

Check the existing resources:

```bash
kubectl get nodes -o wide
kubectl get deployments
kubectl get pods -o wide --show-labels
kubectl get services
```

Make sure the Nginx Pods are ready before continuing.

## 1. Create the NodePort Service YAML

Create a file named `nginx-nodeport-service.yaml`:

```bash
nano nginx-nodeport-service.yaml
```

Add the following manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx-demo
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
      protocol: TCP
```

Save the file in Nano with `Ctrl+O`, press `Enter`, then exit with `Ctrl+X`.

### Explanation

- `apiVersion: v1`: API version for a Service.
- `kind: Service`: creates a Kubernetes Service.
- `metadata.name`: names the Service `nginx-nodeport`.
- `type: NodePort`: exposes the Service on a port of each node.
- `selector.app`: selects Pods labelled `app: nginx-demo`.
- `port: 80`: port exposed by the Service.
- `targetPort: 80`: port on the selected Pods receiving traffic.
- `nodePort: 30080`: port clients use on a reachable node.
- `protocol: TCP`: uses TCP networking.

The usual default NodePort range is `30000–32767`. The requested port must be available and within the cluster's configured NodePort range.

## 2. Validate and apply the manifest

Run a client-side dry run:

```bash
kubectl apply --dry-run=client -f nginx-nodeport-service.yaml
```

Apply the Service:

```bash
kubectl apply -f nginx-nodeport-service.yaml
```

## 3. Verify the Service

```bash
kubectl get svc nginx-nodeport
kubectl describe svc nginx-nodeport
```

Confirm that:

- The Service type is `NodePort`.
- The Service exposes port `80`.
- The NodePort is `30080`.
- The selector is `app=nginx-demo`.
- Backend endpoints are listed.

Check EndpointSlices:

```bash
kubectl get endpointslices -l kubernetes.io/service-name=nginx-nodeport
```

The endpoints should correspond to the ready Nginx Pods.

## 4. Find a node IP address

Run:

```bash
kubectl get nodes -o wide
```

Choose a node IP address that is reachable from the machine where you will run the test.

## 5. Test from the terminal

Replace `<NODE-IP>` with a reachable node IP:

```bash
curl -i http://<NODE-IP>:30080
```

Expected response includes an HTTP success status such as `HTTP/1.1 200 OK` and the Nginx welcome HTML.

For example, if the reachable node IP is `192.168.1.20`, use:

```bash
curl -i http://192.168.1.20:30080
```

Use your actual node IP; the example address is illustrative.

## 6. Test from a browser

If the node IP is reachable from your browser and network/firewall rules allow the connection, open:

```text
http://<NODE-IP>:30080
```

You should see the Nginx Welcome page.

A node IP may be private or reachable only inside a lab or corporate network. If the browser cannot connect, the cause may be routing, firewall rules, security groups, or network isolation. A successful curl from inside a lab does not guarantee that a personal computer outside that network can access the same address.

## 7. Test from inside the cluster

If you also want to confirm internal Service access, create a temporary client Pod with curl:

```bash
kubectl run curl-test \
  --image=curlimages/curl:latest \
  --restart=Never \
  --command -- sleep 1800
```

Wait until it is running:

```bash
kubectl get pods
```

Enter the container:

```bash
kubectl exec -it curl-test -- sh
```

Inside the container, run:

```sh
curl -i http://nginx-nodeport
```

Expected result: an HTTP success response and the Nginx welcome HTML.

Exit the container:

```sh
exit
```

This verifies internal access through the Service DNS name. The node-IP-and-port test in Section 5 is the test for NodePort access.

## 8. Understand the traffic flow

```text
External Client
      |
      | http://<NODE-IP>:30080
      v
Kubernetes Node :30080
      |
      v
NodePort Service :80
      |
      +------> Nginx Pod 1 :80
      +------> Nginx Pod 2 :80
      +------> Nginx Pod 3 :80
```

The Service selector identifies the backend Pods. Kubernetes networking directs traffic to an eligible backend.

## 9. Troubleshooting

### Service has no endpoints

```bash
kubectl get pods --show-labels
kubectl describe svc nginx-nodeport
kubectl get endpointslices -l kubernetes.io/service-name=nginx-nodeport
```

Check that the Service selector matches the Pod labels and that the Pods are ready.

### NodePort cannot be reached

1. Confirm the Service exists and uses the expected NodePort:

   ```bash
   kubectl get svc nginx-nodeport
   ```

2. Check node addresses:

   ```bash
   kubectl get nodes -o wide
   ```

3. Test from a machine that can reach the node network.
4. Check routing, firewall rules, security groups, and network policies as applicable.
5. Verify that the selected port is permitted by the cluster configuration.

A browser timeout does not by itself prove the Service is misconfigured.

## 10. Quick command reference

```bash
kubectl get nodes -o wide
kubectl get pods -o wide --show-labels

kubectl apply --dry-run=client -f nginx-nodeport-service.yaml
kubectl apply -f nginx-nodeport-service.yaml

kubectl get svc nginx-nodeport
kubectl describe svc nginx-nodeport
kubectl get endpointslices -l kubernetes.io/service-name=nginx-nodeport

curl -i http://<NODE-IP>:30080
```

## 11. Result checklist

- [ ] The NodePort YAML was created and applied.
- [ ] The Service shows type `NodePort`.
- [ ] The Service selects the Nginx Pods.
- [ ] EndpointSlices show the backend Pod addresses.
- [ ] The `curl` test through a reachable node IP and NodePort succeeds.
- [ ] Browser access was tested where network reachability permits it.
- [ ] Any network limitation was documented accurately.

## 12. Interview takeaway

> A NodePort Service exposes a Kubernetes Service on a port of each node. Clients can access the application using a reachable node IP and the assigned NodePort. The Service forwards traffic to eligible Pods selected by labels. External access still depends on network routing and firewall configuration.

## Next step

Compare this NodePort Service with the `ClusterIP` Service, then create a `LoadBalancer` Service and check whether your Kubernetes platform provisions an external address.
