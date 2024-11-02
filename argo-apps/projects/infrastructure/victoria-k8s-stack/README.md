# victoria-k8s-stack
Installed following https://docs.victoriametrics.com/helm/victoriametrics-k8s-stack, this provides an easy solution to get started monitoring a kubernetess cluster.

This installs Grafana, VictoriaMetrics, the VictoriaMetrics alerting stack, node-exporter, and other tools which I would have installed anyway.  I prefer the VictoriaMetrics flavors of Prometheus tools to Prometheus itself, in my experience they're a little easier to use and lighter on resources.  I still prefer Loki for logging, but this stack does not include logging.