# Kong API Gateway/Ingress Controller
The Kong yaml files in this folder were adapted from the instructions available at https://docs.konghq.com/kubernetes-ingress-controller/latest/get-started/

## CRDs
CRDs yaml created from https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml

## Gateway Objects
Gateway yaml created from Kong instructions:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: kong
  annotations:
    konghq.com/gatewayclass-unmanaged: 'true'

spec:
  controllerName: konghq.com/kic-gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: kong
spec:
  gatewayClassName: kong
  listeners:
  - name: proxy
    port: 80
    protocol: HTTP
```

## Helm
Helm chart yaml adapted from Kong instructions:
```bash
helm repo add kong https://charts.konghq.com
helm repo update
helm install kong kong/ingress -n kong --create-namespace
```

## Creating Ingress Objects
Using the Ingress Controller, you can create Ingress objects for your services like so:
```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: <service-route> # Name of Ingress
      namespace: <service-namespace> # Namespace of the Service
      annotations:
        cert-manager.io/cluster-issuer: cloudflare-issuer # If you are using Cert Manager and a cluster issuer, put it here
    spec:
      ingressClassName: kong
      tls:
      - secretName: <service-cert> # Name of the automatically created SSL certificate
        hosts:
        - <service>.apps.k8s.vdude.io # DNS name of the service
      rules:
      - host: <service>.apps.k8s.vdude.io
        http:
          paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
            service:
              name: <service-name> # The service that you want to access
              port:
                number: 80 # Port as applicable
```