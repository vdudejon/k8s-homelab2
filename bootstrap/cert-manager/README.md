# Cert-Manager
I am using cert-manager with Cloudflare DNS.  The initial bootstrap **does not** contain my API key.  After bootstrap, you must create a secret called `cloudflare-api-token` with your API key

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

In the past, I've used Bitnami Sealed Secrets to keep this in my git repo, but because this is such an early requirement I think it's easier to just do it the once.  In a prod setup, I would probably use a combination of Ansible Vault and Sealed Secrets.
