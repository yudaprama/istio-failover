# Istio Failover

This is example to do failover from data center A to data center B

Use kind to create 2 kubernetes cluster for data center A and data center B on local notebook

```
CTX_DC_A="dc-a"
CTX_DC_B="dc-b"

for CTX in "$CTX_DC_A" "$CTX_DC_B"; do
    kind create cluster --name $CTX
done

for CTX in "$CTX_DC_A" "$CTX_DC_B"; do
    kubectl config use-context "kind-$CTX"
    istioctl install --skip-confirmation --set profile=demo
done

for CTX in "$CTX_DC_A" "$CTX_DC_B"; do
    kubectl config use-context "kind-$CTX"
    kubectl apply --filename sample.yaml
    kubectl apply --namespace sample --filename "helloworld-$CTX.yaml"
done

kubectl config use-context "kind-$CTX_DC_A"
kubectl apply --namespace sample --filename sleep.yaml

CTX_PRIMARY="$CTX_DC_A"
kubectl config use-context "kind-$CTX_PRIMARY"
kubectl apply --namespace sample --filename destinationrule.yaml

kubectl exec deployments/sleep \
    --namespace sample \
    -- curl -sSL http://helloworld.sample:5000/hello

kubectl exec deployments/helloworld-dc-a \
    --namespace sample \
    --container istio-proxy \
    -- curl -sSL -X POST http://127.0.0.1:15000/drain_listeners

for CTX in "$CTX_DC_A" "$CTX_DC_B"; do
    kind cluster delete "$CTX"
done
```
