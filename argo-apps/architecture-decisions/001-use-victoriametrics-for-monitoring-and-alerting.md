---
# These are optional metadata elements. Feel free to remove any of them.
status: "accepted"
date: "2024-11-02"
---

# Use VictoriaMetrics for Monitoring and Alerting

## Context and Problem Statement

I need a robust monitoring and alerting solution for my homelab and home. Currently, Prometheus is widely adopted, but I wanted to explore alternatives for better performance and efficiency.

<!-- This is an optional element. Feel free to remove. -->
## Decision Drivers

* Resource efficiency: Lower CPU and memory usage
* Ease of use: Simpler configuration and maintenance
* Alerting Flexibility

## Considered Options

* VictoriaMetrics and VMAlert
* Prometheus and AlertManager

## Decision Outcome

Chosen option: VictoriaMetrics and VMAlert

<!-- This is an optional element. Feel free to remove. -->
### Consequences

* Good, because lower CPU and memory usage compared to Prometheus, simpler configuration
* Bad, because Prometheus community support is much larger, documentation is less widely available
* Neutral, learning curve is about the same for both, as the VM stack is fully compatible with Promstack

<!-- This is an optional element. Feel free to remove. -->
## More Information
* VictoriaMetrics docs: https://docs.victoriametrics.com/
* Prometheus docs: https://prometheus.io/docs/