# Service and load-balancing test

The reusable `manifests/nginx-demo.yaml` manifest creates a three-replica nginx Deployment and exposes its Pods through a ClusterIP Service named `nginx-demo`.

```bash
kubectl apply -f manifests/nginx-demo.yaml
kubectl rollout status deployment/nginx-demo
kubectl get pods -l app=nginx-demo -o wide
kubectl get service nginx-demo
kubectl get endpointslice -l kubernetes.io/service-name=nginx-demo
```

The EndpointSlice should contain the ready Pod addresses. Test cluster DNS and the Service from a temporary curl Pod:

```bash
kubectl run curl-test --rm -it --restart=Never \
  --image=curlimages/curl -- curl -s http://nginx-demo
```

To make load distribution visible, give each nginx Pod a unique index page:

```bash
for pod in $(kubectl get pods -l app=nginx-demo -o name); do
  kubectl exec "$pod" -- sh -c "echo Served-by-\$(hostname) > /usr/share/nginx/html/index.html"
done
```

Send repeated requests from inside the cluster:

```bash
kubectl run curl-loop --rm -it --restart=Never --image=curlimages/curl -- \
  sh -c 'for i in 1 2 3 4 5 6 7 8 9; do curl -s http://nginx-demo; done'
```

Responses should show different Pod hostnames. This demonstrates that Service traffic is distributed across ready endpoints; it does not guarantee strict round-robin ordering.

## Cleanup

```bash
kubectl delete -f manifests/nginx-demo.yaml
```
