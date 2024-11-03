# vault

I think the redirect address in the `values.yaml` is going to be a problem if you tried to deploy this in multiple sites.  Will have to do further research to figure that out.

## Initialize Vault
Once Vault is installed for the first time, you gotta get it up and running
1. Initialize vault-0:
```
kubectl exec vault-0 -n vault -- vault operator init \
    -key-shares=10 \
    -key-threshold=3 \
    -format=json > cluster-keys.json
```
2. Display the unseal key found in cluster-keys.json with `jq -r ".unseal_keys_b64[]" cluster-keys.json`

```
Warning: Do not run an unsealed Vault in production with a single key share and a single key threshold.
Above code uses 10 key shares and threshold 3
```
3. Unseal vault-0 using 3 of the keys:
```
kubectl exec vault-0 -n vault -- vault operator unseal <KEY1>
kubectl exec vault-0 -n vault -- vault operator unseal <KEY2>
kubectl exec vault-0 -n vault -- vault operator unseal <KEY3>
```
4. Join vault-1 and vault-2 to the cluster
```
kubectl exec -ti vault-1 -n vault -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -ti vault-2 -n vault -- vault operator raft join http://vault-0.vault-internal:8200
```
5. Unseal vault-1 and vault-2, repeating step 3 but against vault-1 and vault-2
```
kubectl exec vault-1 -n vault -- vault operator unseal <KEY1>
kubectl exec vault-1 -n vault -- vault operator unseal <KEY2>
kubectl exec vault-1 -n vault -- vault operator unseal <KEY3>
kubectl exec vault-2 -n vault -- vault operator unseal <KEY1>
kubectl exec vault-2 -n vault -- vault operator unseal <KEY2>
kubectl exec vault-2 -n vault -- vault operator unseal <KEY3>
```

## Configure Kubernetes Auth
1. Log in to the UI and go to Access > Authentication Methods > Enable New Method
2. Choose Kubernetes
3. Click Enable Method
4. Set the host to https://kubernetes.default.svc and click Save
5. For k8s to access the Vault secrets, you need to create a k8s service account and create a kubernetes authentication role in Vault that binds the role to the service account.  
    > **Note:** For my homelab, I'm going to create a service account with read access to everything in Vault
6. In the Vault UI, click on the kubernetes authentication method, and then click Create Role
    - Role Name: vault-read-all
    - Bound service account name: vault-read-all-account
    - Bound service account namespaces: *
7. Create a new kv engine called kubernetes-secrets
8. Create a new ACL policy named vault-read-all (this needs to match the Role name from earlier)
    ```json
    path "kubernetes-secrets" {
        capabilities = ["read"]
    }
    ```
9. In kubernetes, create the service account named vault-read-all-account
    ```bash
    kubectl -n vault create serviceaccount vault-read-all-account
    ```

Finish following the guide here https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-sidecar