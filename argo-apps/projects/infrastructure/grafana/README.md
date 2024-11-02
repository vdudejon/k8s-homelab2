# Grafana
## UNUSED
This is actually unused and omitted from `projects/infrastructure/apps/overlays/dev/kustomization.yaml` in favor of the VictoriaMetrics K8s Stack, but I'm leaving it here for future reference.

## Get Admin Password
```bash
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```