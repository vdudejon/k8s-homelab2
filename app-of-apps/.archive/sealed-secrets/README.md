# Bitnami Sealed Secrets


## Install kubeseal Client
```bash
sudo dnf install curl jq -y
KUBESEAL_VERSION=$(curl -s https://api.github.com/repos/bitnami-labs/sealed-secrets/tags | jq -r '.[0].name' | cut -c 2-)
curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz"
tar -xvzf kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

## Converting Secrets from Files
If you have an existing k8s secret in a file, for example your Portworx secret, you can convert it to a Sealed Secret quite easily.  
1. Save the secret to a file where `kubeseal` is installed
2. Run the following, which will save the Sealed Secret to `sealedsecret.yaml`:
    ```bash
    kubeseal --controller-name sealed-secrets --controller-namespace sealed-secrets kubeseal -f secret.yaml -w sealedsecret.yaml
    ```
3. You can safely place the sealedsecret.yaml file into GitHub.
4. Delete `secret.yaml`