# AWX

For the AWX instance to deploy, the operator must exist in the same namespace.

Ran into a weird issue where the pgsql server wouldn't deploy, it had errors like `cannot create directory '/var/lib/pgsql/data/userdata': Permission denied" `.  Followed instructions from this issue: https://github.com/ansible/awx-operator/issues/1770.  In the AWX yaml, added:
```yaml
    spec:
      postgres_data_volume_init: true
      postgres_init_container_commands: |
        chown 26:0 /var/lib/pgsql/data
        chmod 700 /var/lib/pgsql/data