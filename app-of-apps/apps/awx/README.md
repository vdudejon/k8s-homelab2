# AWX

For the AWX instance to deploy, the operator must exist in the same namespace.

Couldn't get the ingress to createa cert using the manifest, so I just explicitly created one.

Ran into a weird issue where the pgsql server wouldn't deploy, it had errors like `cannot create directory '/var/lib/pgsql/data/userdata': Permission denied" `.  Followed instructions from this issue: https://github.com/ansible/awx-operator/issues/1770.  In the AWX yaml, added:
```yaml
    spec:
      postgres_data_volume_init: true
      postgres_init_container_commands: |
        chown 26:0 /var/lib/pgsql/data
        chmod 700 /var/lib/pgsql/data
```

By default, the admin user is admin and the password is available in the <resourcename>-admin-password secret. To retrieve the admin password, run:
```bash
kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode ; echo
<password displayed here>
```