# vault-secrets-operator

Following the guide from Hashicorp here: https://developer.hashicorp.com/vault/tutorials/kubernetes/vault-secrets-operator which references the repo here: https://github.com/hashicorp-education/learn-vault-secrets-operator/tree/main

## Install Vault CLI
A lot of things can't be configured using the UI.  
1. Install the Vault CLI on Red Hat:
```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install vault
```
2. Export the vault address as an environmental variable
```bash
export VAULT_ADDR='https://vault.apps.k8s.vdude.io'
```
3. Login to vault
```bash
vault login
```

## Configure Vault 
1. Create a kvv2 secrets engine for your secrets
```bash
vault secrets enable -path=kubernetes-secrets kv-v2
```
2. Create a Vault Policy with access to the secrets.  I am being lazy/less safe and settings things up such that my kubernetes service account will be able to see all secrets.
```bash
tee k8s-read-all.json <<EOF
path "kubernetes-secrets/*" {
   capabilities = ["read", "list"]
}
EOF
vault policy write k8s-read-all k8s-read-all.json
```
3. Enable the Kubernetes auth method and configure for your kubernetes cluster
```bash
vault auth enable -path k8s-auth-mount kubernetes
vault write auth/k8s-auth-mount/config kubernetes_host="https://kubernetes.devault.svc:443"
```
4. Create a Vault Role on the kubernetes auth mount that associates a service account to the policy. In my example, I will allow any service account named `vault-auth` in any namespace read access to all secrets.  Again, this is not particularly secure.
```bash
vault write auth/k8s-auth-mount/role/k8s-read-all \
   bound_service_account_names=vault-auth \
   bound_service_account_namespaces="*" \
   policies=k8s-read-all \
   audience=vault \
   ttl=24h
```