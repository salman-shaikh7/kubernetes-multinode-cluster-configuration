# Validation evidence

Use this checklist after building the cluster. The snippets below describe the expected state; replace them with output captured from the running lab when publishing evidence.

## 1. Nodes

Run on the control plane:

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get svc,endpointslice
```

Confirm that all three nodes report `Ready` and use the expected internal addresses:

| Node | Expected address |
|---|---|
| `k8s-control-plane` | `192.168.122.131` |
| `k8s-worker-1` | `192.168.122.94` |
| `k8s-worker-2` | `192.168.122.38` |

The node list should have this overall state:

```text
k8s-control-plane   Ready   control-plane
k8s-worker-1        Ready   <none>
k8s-worker-2        Ready   <none>
```

## 2. Control plane and Calico

```bash
kubectl get pods -n kube-system -o wide
kubectl get tigerastatus
kubectl get pods -n calico-system -o wide
```

Confirm that the control-plane components are running and every Tigera status component is available.

## 3. Workload placement

```bash
kubectl apply -f manifests/nginx-demo.yaml
kubectl rollout status deployment/nginx-demo
kubectl get pods -l app=nginx-demo -o wide
```

Confirm that three nginx Pods are ready. Record their node and Pod-IP columns to demonstrate scheduling and Calico address allocation.

## 4. Service endpoints and DNS

```bash
kubectl get deployment,pods,service -l app=nginx-demo
kubectl get endpointslice -l kubernetes.io/service-name=nginx-demo
kubectl run curl-test --rm -it --restart=Never \
  --image=curlimages/curl -- curl -s http://nginx-demo
```

Confirm that the EndpointSlice contains the ready Pod addresses and that the temporary Pod resolves and reaches `nginx-demo`.

## 5. Load distribution

Follow the repeated-request test in [Service and load-balancing test](08-service-load-balancing.md). Capture output showing responses from more than one Pod.

## Publishing screenshots

Real screenshots make the project easier to verify. Place sanitized images in `assets/screenshots/` with descriptive names such as:

- `nodes-ready.png`
- `calico-status.png`
- `nginx-pod-placement.png`
- `service-endpoints.png`
- `load-distribution.png`

Do not publish bootstrap tokens, certificate material, kubeconfig contents, private keys, or unrelated terminal history. After adding screenshots, embed them in this document with relative Markdown paths.
