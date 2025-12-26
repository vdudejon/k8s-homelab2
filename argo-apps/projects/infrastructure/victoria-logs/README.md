# victoria-k8s-stack
Installed following https://docs.victoriametrics.com/helm/victoriametrics-k8s-stack, this provides an easy solution to get started monitoring a kubernetess cluster.

This installs Grafana, VictoriaMetrics, the VictoriaMetrics alerting stack, node-exporter, and other tools which I would have installed anyway.  I prefer the VictoriaMetrics flavors of Prometheus tools to Prometheus itself, in my experience they're a little easier to use and lighter on resources.  I still prefer Loki for logging, but this stack does not include logging.

## Notes
Keep your release name short, or some of the object names will be longer than 63 characters and cause errors.  Release name victoria-k8s-stack made one 64-chars long, but vm-k8s was fine.