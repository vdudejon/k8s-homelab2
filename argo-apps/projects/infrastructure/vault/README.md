# vault

I think the redirect address in the `values.yaml` is going to be a problem if you tried to deploy this in multiple sites.  Will have to do further research to figure that out.

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
