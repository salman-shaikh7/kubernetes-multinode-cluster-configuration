# Service and load-balancing test

This test connects several core Kubernetes ideas:

- A Deployment maintains three nginx replicas and replaces a Pod if one fails.
- The scheduler can place those Pods on available worker nodes.
- A ClusterIP Service provides one stable virtual address and DNS name even though Pod addresses can change.
- An EndpointSlice records the ready Pods currently backing that Service.

## Deploy and inspect the workload

The reusable `manifests/nginx-demo.yaml` manifest creates the Deployment and Service:

```bash
kubectl apply -f manifests/nginx-demo.yaml
kubectl rollout status deployment/nginx-demo
kubectl get pods -l app=nginx-demo -o wide
kubectl get service nginx-demo
kubectl get endpointslice -l kubernetes.io/service-name=nginx-demo
```

The rollout command waits for all three replicas to become available. The wide Pod view reveals which node runs each replica, and the EndpointSlice should contain their ready Pod addresses.

## Test Service discovery

Create a temporary Pod inside the cluster and request the Service by name:

```bash
kubectl run curl-test --rm -it --restart=Never \
  --image=curlimages/curl -- curl -s http://nginx-demo
```

This checks two separate mechanisms: cluster DNS must resolve `nginx-demo`, and Service routing must deliver the request to a ready nginx Pod. The temporary Pod is deleted automatically when the command finishes.

## Make load distribution visible

Give each nginx Pod a unique index page containing its hostname:

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

Responses should show different Pod hostnames. This demonstrates that Service traffic is distributed across ready endpoints; it does not guarantee strict round-robin ordering because the implementation chooses endpoints per connection and can vary by networking mode.

## Cleanup

Deleting the manifest removes both the Deployment and Service created by the demo, which also causes Kubernetes to terminate the managed Pods:

```bash
kubectl delete -f manifests/nginx-demo.yaml
```
