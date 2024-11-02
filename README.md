# k8s-homelab2
My second k3s homelab, because OpenShift is too hungry

The goal is to install k3s in HA mode on 3 servers, using Kube VIP to load balance them.  Portworx Community Edition will be used for container storage.  NFS will be used for large/long-term storage.  Kong Ingress Controller for Ingress.  ArgoCD for as much past initial configuration as possible.

## Prep
### DNS
I'm copying the OpenShift pattern of having a subdomain for the cluster, a VIP for the API/control plane, and a wildcard VIP for apps that the Ingress Controller will use.  To follow this, you will need to:
1. Decide on a name/subdomain for your cluster.
    - In my case, my domain is `vdude.io`, my cluster name is `k8s`, so my cluster subdomain is `k8s.vdude.io`
2. Assign a static IP and DNS record for the API
    - I chose `api.k8s.vdude.io` at 10.1.1.230
3. Assign a static IP and wildcard DNS entry for apps
    - I chose `*.apps.k8s.vdude.io` at 10.1.1.231
4. (Optional) Assign a range for Load Balancer use
    - I chose 10.1.1.232-10.1.1.240
    - When using these, you will need to create DNS entries manually for whatever service uses the IP

### Nodes
1. Log in to each node at switch to the root user `sudo su -`
2. On all nodes, disable firewalld
    ```bash
    systemctl disable firewalld --now
    ```
3. On the server you use for command line tools (node 1, a jump box, your laptop), install Helm
    ```bash
    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    ```

### Create Manifests Folder, Upload Bootstrap Folder
K3s has an optional manifests directory that will be searched to auto-deploy any manifests found within. 

1. On the first node, create this directory first in order to later place the bootstrap resources inside.  
    ```bash 
    mkdir -p /var/lib/rancher/k3s/server/manifests
    ```

2. Copy the `bootstrap` folder to the manifests folder `/var/lib/rancher/k3s/server/manifests`.  This will install Kube-VIP and Kong Ingress Controller

## Install HA K3S Cluster
1. On node 1, create a token
    ```bash
    export TOKEN=$(openssl rand -base64 48)
    export TOKEN='<paste from above>'
    ```
2. On nodes 2 & 3, export the token variable
    ```bash
    export TOKEN='<match node 1>'

3. Run the k3s installer command on the first node:
    ```bash
    curl -sfL https://get.k3s.io | K3S_TOKEN=$TOKEN sh -s - server \
    --cluster-init \
    --disable servicelb \
    --disable traefik \
    --tls-san=10.1.1.230
    ```

4. Run the k3s installer on nodes 2 and 3
    ```bash
    curl -sfL https://get.k3s.io | K3S_TOKEN=$TOKEN sh -s - server \
    --server https://10.1.1.211:6443 \
    --disable servicelb \
    --disable traefik \
    --tls-san=10.1.1.230
    ```

5. The kubeconfig file is stored in `/etc/rancher/k3s/k3s.yaml`, you will need to change the server ip to the VIP IP if you copy it down to your own computer or what not.  Make it readable for non-root users with:
    ```bash
    sudo chmod 600 /etc/rancher/k3s/k3s.yaml
    ```

This will get you as far as a kubernetes cluster with an HA API endpoint, the Kong Ingress Controller, and cert-manager with a Cloudflare cluster issuer (without an API key).  For cert-manager to work, you will need to create a secret with your Cloudflare API key.

### Finish Cert Manager Setup by Creating Cloudflare Secret
Get your API key from Cloudflare, and then add it to a secret that Cert Manager can use:

    ```bash
    echo "
    apiVersion: v1
    kind: Secret
    metadata:
        name: cloudflare-api-token
        namespace: cert-manager
    type: Opaque
    stringData:
        api-token: <Cloudflare API token>
    " | kubectl apply -f -
    ```


### Install ArgoCD
I tried to put this in the bootstrap but had a lot of trouble.  So let's install it semi-manually using Helm.

### Important Notes
- Since we are putting SSL certs at the Ingress Controller, TLS must be disabled on ArgoCD.  Using helm, this is specified in the values file.  If you use a manifest, you need to edit the container args after it is deployed like this:
    ```bash
    kubectl -n argocd edit deployment argocd-server
    ```
    Under `args` add `- --insecure`:
    ```yaml
    ...
    containers:                                                     
    - args:
      - /usr/local/bin/argocd-server
      - --insecure # ADD THIS LINE
      env:
        ...
    ```
- For ArgoCD to use Kustomize with Helm, you need to update the config map after it is deployed.  This is in `argo-apps\projects\infrastructure\argo-cd\base\patch-configmap.yaml`
## Installation
1. Copy `values.yaml` and `patch-configmap.yaml` from `argo-apps\projects\infrastructure\argo-cd\base` to a machine that has Helm installed on it
2. Run:
    ```bash
    helm repo add argo https://argoproj.github.io/argo-helm
    helm install argo-cd argo/argo-cd \
    --version 7.6.12 \
    -n argo-cd --create-namespace \
    -f values.yaml
    ```
3. After it is installed, run 
    ```bash
    kubectl -n argocd apply -f patch-configmap.yaml
    ```
4. Create an Ingress by copying the `ingress.yaml` file from `argo-apps\projects\infrastructure\argo-cd\overlays\dev`, or apply like so:
    ```bash
    echo '
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
        name: argocd-route
        namespace: argocd
        annotations:
            cert-manager.io/cluster-issuer: cloudflare-issuer
    spec:
        ingressClassName: kong
        tls:
        - secretName: argocd-cert
          hosts:
          - argocd.apps.k8s.vdude.io
        rules:
        - host: argocd.apps.k8s.vdude.io
          http:
            paths:
            - path: /
              pathType: ImplementationSpecific
              backend:
                service:
                    name: argocd-server
                    port:
                        number: 80
    ' | kubectl apply -f -
    ```

5. Get the initial password
    ```bash
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode; echo
    ```

6. Log in to ArgoCD using user `admin` and the password from above at https://argocd.apps.k8s.vdude.io

7. Configure the repo, then add in the `argo-apps`

8. ???

9. Profit