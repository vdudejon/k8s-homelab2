# authentik

authentik (with a lowercase a) is a Swiss Army Knife of an authentication provider.

For authentik to work behind a reverse proxy, such as I am using with Kong, you need to specify specific headers to be forwarded.  See `overlays\dev\ingress.yaml`

### Configuring Ansible AWX
Carefully follow the guide here: https://docs.goauthentik.io/integrations/services/awx-tower/
**NOTE** Make sure you set a signing certificate under "Advanced Protocol Settings."  This can be the default self-signed cert.  ChatGPT also suggests the following, which might be done automatically when you select a signing cert because I didn't explicitly set them:

    Auth providers such as AWX typically require that the SAML assertions (or responses) be signed to verify the authenticity of the data. To enable signing in Authentik:

        1. Go to your Authentik SAML provider configuration.
        2. Ensure that "Sign AuthnResponse" and "Sign Assertions" are enabled.
        3. If there’s an option for "Sign Metadata", enable it as well to ensure that the metadata provided to AWX is signed, which improves compatibility.

### Configuring Grafana
Follow the guide here: https://docs.goauthentik.io/integrations/services/grafana/.  The guide actually uses the configuration file, but I just did it through the UI and figured out which setting was which based on the setting names.  

For this to work, you need to have the grafana.ini.server.domain and root_url set.  This can be done using `values.yaml` but that kind of breaks deploying into multiple environments.  There may be a better solution, possibly using environmental variables.

I couldn't get roles syncing to work at first, and as a workaround I disabled syncing roles and made myself an administrator after logging in once.  To bypass the SSO auth, you need disable Auto-Login otherwise you won't be able to get back in as a local user.  Once you set yourself as an admin, you can turn auto login back on.

I realized later I had a trailing " mark at the end of the role sync line.  Removing that made it work as expected.  You also need to enable the setting Allow assign Grafana Admin.
`contains(groups, 'authentik Admins') && 'Admin' || contains(groups, 'Grafana Editors') && 'Editor' || 'Viewer'`

### Configuring Vault
Follow the guide here: https://docs.goauthentik.io/integrations/services/hashicorp-vault/. 

To use the Vault client, you can terminal to one of the Vault pods, or install Vault on your own machine.  You can use the GUI to set the config, but not the role.

```bash
kubectl exec -it vault-0 -n vault -- /bin/sh
vault login 
vault write auth/oidc/role/reader \
      bound_audiences="<Client ID>" \
      allowed_redirect_uris="https://vault.apps.k8s.vdude.io/ui/vault/auth/oidc/oidc/callback" \
      allowed_redirect_uris="https://vault.apps.k8s.vdude.io/oidc/callback" \
      allowed_redirect_uris="http://localhost:8250/oidc/callback" \
      user_claim="sub" \
      policies="reader"
```

Need to learn more on how to grant additional permissions.  