# kube-vip
The yaml files created in this folder were adapated from the instructions at
- https://kube-vip.io/docs/usage/k3s/
- https://kube-vip.io/docs/usage/cloud-provider/#install-the-kube-vip-cloud-provider


## Kube-VIP and Kube-VIP Cloud Provider
Kube-VIP will provide a way for load balancing the kubernetes API/Control Plane.  Kube-VIP Cloud Provider will provide a way to grant the Ingress Controller an external IP.  I'm copying the pattern OpenShift also uses, which is to have 1 VIP for the control plane and 1 VIP for routes/apps.  The plan here is to use 1 ingress controller or API gateway to provide access in, so I created a wildcard DNS entry for apps.k8s.vdude.io.  I'm using Pi Hole for internal DNS.  The end result is that future tools will be deployed as XYZ.apps.k8s.vdude.io, ie grafana.apps.k8s.vdude.io, which will point to the ingress controller.

1. Generate the `rbac.yaml` file
    ```bash
    curl https://kube-vip.io/manifests/rbac.yaml > rbac.yaml
    ```
2. Create the Kube VIP manifest.  I did this from a different system that already had docker installed.  The yaml here can probably just be copied and updated.
    1. Set the VIP IP to be used 
        ```bash
        export VIP=10.1.1.230
        ```
    2. Set the INTERFACE name to be the interface on the servers that will be used.  Use `ip a` to find it.
        ```bash
        export INTERFACE=enp0s20f0u3
        ```
    3. Get the latest version of the kube-vip release by parsing the GitHub API
        ```bash
        export KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")
        ```
    4. If the system has Docker installed, set this alias
        ```bash
        alias kube-vip="docker run --network host --rm ghcr.io/kube-vip/kube-vip:$KVVERSION"
        ```
    5. Create the DaemonSet yaml:
        ```bash
        kube-vip manifest daemonset \
            --interface $INTERFACE \
            --address $VIP \
            --inCluster \
            --taint \
            --controlplane \
            --services \
            --arp \
            --leaderElection
        ```
3. Save this as `arp-ds.yaml`
4. Get the Cloud Provider yaml:
    ```bash
    curl https://raw.githubusercontent.com/kube-vip/kube-vip-cloud-provider/main/manifest/kube-vip/kube-vip-cloud-controller.yaml > cloud-provider.yaml
    ```
5. Create a ConfigMap with your VIP IP or range if you want to create multiple Load Balancers in the future.  Save to `cloud-provider-configman.yaml`:
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: kubevip
    namespace: kube-system
    data:
        range-kong: 10.1.1.231-10.1.1.231
        range-global: 10.1.1.232-10.1.1.240
    ```
    To reserve specific IPs for specific namespaces, set them as `range-<namespace>` as in `range-kong` above