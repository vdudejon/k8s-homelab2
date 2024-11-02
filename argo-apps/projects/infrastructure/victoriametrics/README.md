# Victoria Metrics
## UNUSED
This is actually unused and omitted from `projects/infrastructure/apps/overlays/dev/kustomization.yaml` in favor of the VictoriaMetrics K8s Stack, but I'm leaving it here for future reference.

## Add to Grafana
You can add Victoria to Grafana by adding a Prometheus datasource and point to `http://victoriametrics-victoria-metrics-single-server.victoriametrics.svc:8428`.  There is no auth by default.

