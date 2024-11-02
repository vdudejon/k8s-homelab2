# Kuma

## Using Kong as a Delegated Gateway
Label the Kong namespace 
```bash
kubectl label namespace kong kuma.io/sidecar-injection=enabled
```
Restart both the controller and the gateway to leverage sidecar injection:
```bash
kubectl rollout restart -n kong deployment kong-gateway kong-controller
```
Wait until pods are fully rolled out and look at them:
```bash
kubectl get pods -n kong
```
It is now visible that both pods have 2 containers, one for the application and one for the sidecar.
```bash
NAME                              READY   STATUS    RESTARTS      AGE
kong-controller-675d48d48-vqllj   2/2     Running   2 (69s ago)   72s
kong-gateway-674c44c5c4-cvsr8     2/2     Running   0             72s
```