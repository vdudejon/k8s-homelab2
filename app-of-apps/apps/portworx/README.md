# Portworx
Ensure the Portworx Operator is installed, see `app-of-apps/operators/portworx.`

To install Portworx Community Edition, you will need to go to Portworx Central and create the manifest using their wizard.

Once you have the manifest, convert the secret part into a sealed secret:
1. Save the secret to a file where `kubeseal` is installed
2. Run the following, which will save the Sealed Secret to `sealedsecret.yaml`:
    ```bash
    kubeseal --controller-name sealed-secrets --controller-namespace sealed-secrets kubeseal -f secret.yaml -w sealedsecret.yaml
    ```
3. You can safely place the sealedsecret.yaml file into GitHub.
4. Delete `secret.yaml`

Paste the `StorageCluster` manifest into `portworx.yaml`