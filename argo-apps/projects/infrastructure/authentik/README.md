# authentik

authentik (with a lowercase a) is a Swiss Army Knife of an authentication provider.

### Configuring Ansible AWX
Carefully follow the guide here: https://docs.goauthentik.io/integrations/services/awx-tower/
**NOTE** Make sure you set a signing certificate under "Advanced Protocol Settings."  This can be the default self-signed cert.  ChatGPT also suggests the following, which might be done automatically when you select a signing cert because I didn't explicitly set them:

    Auth providers such as AWX typically require that the SAML assertions (or responses) be signed to verify the authenticity of the data. To enable signing in Authentik:

        1. Go to your Authentik SAML provider configuration.
        2. Ensure that "Sign AuthnResponse" and "Sign Assertions" are enabled.
        3. If there’s an option for "Sign Metadata", enable it as well to ensure that the metadata provided to AWX is signed, which improves compatibility.
